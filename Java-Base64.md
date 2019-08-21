---
title: Java-Base64
date: 2018-02-07 13:09:29
categories: 
- Java
tags:
- Java
---

某钱包分享链接泄漏手机号码了，[链接](https://www.v2ex.com/t/429099#reply48)

安全问题很重要啊

<!--more-->

## 简介

BASE64 编码是一种常用的字符编码，在很多地方都会用到。但base64不是安全领域下的加密解密算法。能起到**安全作用的效果很差**，而且很容易破解。

他核心作用应该是传输数据的正确性，有些网关或系统只能使用ASCII字符。Base64就是用来将非ASCII字符的数据转换成ASCII字符的一种方法，而且base64特别适合在http，mime协议下快速传输数据。

***

## Java实现

在Java 8在java.util包下面实现了BASE64编解码API，java.util.Base64

### Basic编码

是标准的BASE64编码，用于处理常规的需求

输出的内容不添加换行符，而且输出的内容由字母加数字组成

```java
// 编码
String asB64 = Base64.getEncoder().encodeToString("some string".getBytes("utf-8"));
System.out.println(asB64); 
// 输出为: c29tZSBzdHJpbmc=

// 解码
byte[] asBytes = Base64.getDecoder().decode("c29tZSBzdHJpbmc=");
System.out.println(new String(asBytes, "utf-8")); 
// 输出为: some string
```

### URL编码

如果是使用基本的编码器，那么输出可能会包含反斜线“/”字符，但是如果使用URL编码器，那么输出的内容对URL来说是安全的。

```java
String urlEncoded = Base64.getUrlEncoder().encodeToString("subjects?abcd".getBytes("utf-8"));
System.out.println("Using URL Alphabet: " + urlEncoded);
// 输出为:Using URL Alphabet: c3ViamVjdHM_YWJjZA==
```

编码标准： [RFC 4648](http://www.ietf.org/rfc/rfc4648.txt)  page7

>```
>Josefsson                   Standards Track                     [Page 7]
>
>RFC 4648                    Base-N Encodings                October 2006
>
>
>         Table 2: The "URL and Filename safe" Base 64 Alphabet
>
>     Value Encoding  Value Encoding  Value Encoding  Value Encoding
>         0 A            17 R            34 i            51 z
>         1 B            18 S            35 j            52 0
>         2 C            19 T            36 k            53 1
>         3 D            20 U            37 l            54 2
>         4 E            21 V            38 m            55 3
>         5 F            22 W            39 n            56 4
>         6 G            23 X            40 o            57 5
>         7 H            24 Y            41 p            58 6
>         8 I            25 Z            42 q            59 7
>         9 J            26 a            43 r            60 8
>        10 K            27 b            44 s            61 9
>        11 L            28 c            45 t            62 - (minus)
>        12 M            29 d            46 u            63 _
>        13 N            30 e            47 v           (underline)
>        14 O            31 f            48 w
>        15 P            32 g            49 x
>        16 Q            33 h            50 y         (pad) =
>
>5.  Base 64 Encoding with URL and Filename Safe Alphabet
>
>   The Base 64 encoding with an URL and filename safe alphabet has been
>   used in [12].
>
>   An alternative alphabet has been suggested that would use "~" as the
>   63rd character.  Since the "~" character has special meaning in some
>   file system environments, the encoding described in this section is
>   recommended instead.  The remaining unreserved URI character is ".",
>   but some file system environments do not permit multiple "." in a
>   filename, thus making the "." character unattractive as well.
>
>   The pad character "=" is typically percent-encoded when used in an
>   URI [9], but if the data length is known implicitly, this can be
>   avoided by skipping the padding; see section 3.2.
>
>   This encoding may be referred to as "base64url".  This encoding
>   should not be regarded as the same as the "base64" encoding and
>   should not be referred to as only "base64".  Unless clarified
>   otherwise, "base64" refers to the base 64 in the previous section.
>
>   This encoding is technically identical to the previous one, except
>   for the 62:nd and 63:rd alphabet character, as indicated in Table 2.
>```



标准的Base64并不适合直接放在URL里传输，因为URL编码器会把标准Base64中的“/”和“+”字符变为形如“%XX”的形式，而这些“%”号在存入数据库时还需要再进行转换，因为ANSISQL中已将“%”号用作通配符。

为解决此问题，可采用一种用于URL的改进Base64编码，**它在末尾填充“=”号，并将标准Base64中的“+”和“/”分别改成了“-”和“_”，**这样就免去了在URL编解码和数据库存储时所要作的转换，避免了编码信息长度在此过程中的增加，并统一了数据库、表单等处对象标识符的格式。

### MIME编码

使用基本的字母数字产生BASE64输出，而且对MIME格式友好：每一行输出不超过76个字符，而且每行以“\r\n”符结束。

```java
StringBuilder sb = new StringBuilder();
for (int t = 0; t < 10; ++t) {
  sb.append(UUID.randomUUID().toString());
}
byte[] toEncode = sb.toString().getBytes("utf-8");
String mimeEncoded = Base64.getMimeEncoder().encodeToString(toEncode);
System.out.println(mimeEncoded);
```

>延伸点：[MIME编码](https://zh.wikipedia.org/wiki/%E5%A4%9A%E7%94%A8%E9%80%94%E4%BA%92%E8%81%AF%E7%B6%B2%E9%83%B5%E4%BB%B6%E6%93%B4%E5%B1%95)



## 参考

[Java 8新特性探究（十一）Base64详解](https://my.oschina.net/benhaile/blog/267738)

[Base64编码原理解析与Java实现](http://www.imooc.com/article/9761)