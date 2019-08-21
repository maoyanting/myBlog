---
title: cookie
date: 2018-09-18 22:30:50
tags:
- cookie
---

今天被我们的前端同学气的半死，但是他问我为什么没有cookie的时候我是没有看见cookie……

恶补一下cookie的知识，为了下次能不要再懵逼

<!--more-->



## Cookie

### 使用

我们一般是这么使用的

先往cookie里放东西

```java
        //new 一个cookie 
        Cookie cookie = new Cookie(name, value);
        //往里面塞东西
        cookie.setDomain(domain);
        cookie.setPath(path);
        //扔出去
        response.addCookie(cookie);
```

需要从cookie拿东西的时候

```java
//从request拿到cookie
Cookie[] cookies = request.getCookies();
//拿到你想要的
if (cookies != null && cookies.length > 0) {
            for (Cookie c : cookies) {
                if (c.getName().equals(name)) {
                    return c;
                }
            }
        }
//一般我们要的最多的就是sessionId
```



普通情况下的使用，看到这里就差不多了。

### 一、限制：

不同的浏览器对于`cookie`的个数和总大小都有限制，不过一般我们正常使用不会超。而且过大每次占用的带宽也会很大。

虽然往`cookie`放很多东西看起来很疯狂，但是在没有出现但在` localStorage` 出现之前，`cookie`被滥用过，当做了存储工具，啥东西都往里放。

### 二、浏览器处理cookie

浏览器会把某个域名的`cookie`储存在浏览器中，再次调用这个域名的时候，就会把这个cookie拿出来，放到`request header `带过去



### 三、浏览器禁止了 cookie

#### 2.1为什么禁止：

有部分浏览器会禁止三方cookie，至于为什么……没查到……

#### 2.2禁用了怎么办：

禁用了cookie，就会导致无法保持会话，毕竟我们的sessionId是靠cookie传递的，有些信息也是放在cookie中。

解决方案就是：**把 Session ID 添加到 URL 信息中** 



### 四、跨域

有时候遇到跨域请求，如果是[非简单跨域](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Access_control_CORS)，需要在正式请求之前先发送options请求，这个时候浏览器不会带着cookie过去，如果有什么需要cookie的拦截器就会返回错误

但是只要cors拦截器通过了，第二个请求还是能正常使用的（要把cors拦截器放在较前面的地方）