---
title: Java-通用代码
date: 2018-03-02 17:27:43
categories: 
- Java
tags:
- Java
---

一些通用的Java代码：从网上下载图片，

<!--more-->

## 下载图片

```java
public class Base{
    public static void main(String[] args) throws Exception {
        URL url = new URL("https://pic1.zhimg.com/v2-db168102e05a711e936be851a685ac3c_r.jpg");
        HttpURLConnection conn = (HttpURLConnection)url.openConnection();
        conn.setRequestMethod("GET");
        conn.setConnectTimeout(5 * 1000);
        InputStream inStream = conn.getInputStream();

        File picImg = new File("gakki.jpg");
        FileOutputStream fileOutputStream = new FileOutputStream(picImg);

        byte[] b = new byte[1024];
        int len = 0;
        while ((len = inStream.read() )!= -1){
            fileOutputStream.write(b,0,len);
        }
        fileOutputStream.close();
    }
}
```

