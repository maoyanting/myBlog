---
title: Java-HttpClient
date: 2018-05-15 23:30:39
categories: 
- Java
tags:
- Java
---



## HttpClient的介绍

### 一般步骤

1. 取得HttpClient对象
2. 封装http请求
3. 执行http请求
4. 处理结果

### 请求类型

GET, HEAD, POST, PUT, DELETE, TRACE 和 OPTIONS

### 官方示例

```java
//1.获得一个httpclient对象
CloseableHttpClient httpclient = HttpClients.createDefault();
//2.生成一个get请求
HttpGet httpget = new HttpGet("http://localhost/");
//3.执行get请求并返回结果
CloseableHttpResponse response = httpclient.execute(httpget);
try {
//4.处理结果
} finally {
//5.关闭response
    response.close();
}
```

<!--more-->

## HTTP request

拼接request

```java
URI uri = new URIBuilder()
        .setScheme("http")
        .setHost("www.google.com")
        .setPath("/search")
        .setParameter("q", "httpclient")
        .setParameter("btnG", "Google Search")
        .setParameter("aq", "f")
        .setParameter("oq", "")
        .build();
HttpGet httpget = new HttpGet(uri);
```



## 发送HttpPost/HttpGet

举例：HttpGet请求，HttpPost类似

```java
 public static String sendGet(String url) {
        //1.获得一个httpclient对象
        CloseableHttpClient httpclient = HttpClients.createDefault();
        //2.生成一个get请求
        HttpGet httpget = new HttpGet(url);
        CloseableHttpResponse response = null;
        //3.执行get请求并返回结果
        try {
            response = httpclient.execute(httpget);
        } catch (IOException e1) {
            e1.printStackTrace();
        }
        String result = null;
        //4.处理结果，这里将结果返回为字符串
        try {
            HttpEntity entity = response.getEntity();
            if (entity != null) {
                //内部使用完entity的流后，关闭流
                result = EntityUtils.toString(entity);
            }
        } catch (ParseException | IOException e) {
            e.printStackTrace();
        } finally {
            try {
                //注意关闭response
                response.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        return result;
    }
```



## 带参数的HttpPost

HttpClient通过UrlEncodedFormEntity/StringEntity，来提交带参数的请求

UrlEncodedFormEntity：需要`List<? extends NameValuePair>`或者`Iterable<? extends NameValuePair>`

```java
public interface NameValuePair {
    String getName();
    String getValue();
}
```

StringEntity：任意的字符串 (推荐使用)

```java
    public static String sendPost(String url, Map<String, String> map) {
        //1.获得一个httpclient对象
        CloseableHttpClient httpclient = HttpClients.createDefault();
        List<NameValuePair> formparams = new ArrayList<NameValuePair>();
        for (Map.Entry<String, String> entry : map.entrySet()) {
            //给参数赋值
            formparams.add(new BasicNameValuePair(entry.getKey(), entry.getValue()));
        }
        UrlEncodedFormEntity entity = new UrlEncodedFormEntity(formparams, Consts.UTF_8);
        //2.生成一个post请求
        HttpPost httppost = new HttpPost(url);
        //3.放入参数
        httppost.setEntity(entity);
        
        //4.执行get请求并返回结果
        CloseableHttpResponse response = null;
        try {
            response = httpclient.execute(httppost);
        } catch (IOException e) {
            e.printStackTrace();
        }
        //5.处理结果，这里将结果返回为字符串
        String result = null;
        try {
            HttpEntity entity = response.getEntity();
            if (entity != null) {
                //内部使用完entity的流后，关闭流
                result = EntityUtils.toString(entity);
            }
        } catch (ParseException | IOException e) {
            e.printStackTrace();
        }finally {
            try {
                //注意关闭response
                response.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        return result;
    }
```

> 用到的jar包有httpclient-4.5.1.jar，httpcore-4.4.3.jar，依赖的jar有commons-logging-1.2.jar 
>
> 注意是Apache HttpClient，不是commons-httpclient



## Response handlers

在处理结果时，我们不但要记得关闭response，还需要记得关闭response中entity的流。这种形式上的要求总是不安全的，一旦我们在编码中忘记了这两个步骤，就会导致连接没有被良好地复用等问题。所以在处理结果时，推荐使用`ResponseHandler`。 

```java
CloseableHttpClient httpclient = HttpClients.createDefault();
HttpGet httpget = new HttpGet("http://localhost/json");

ResponseHandler<MyJsonObject> rh = new ResponseHandler<MyJsonObject>() {
    @Override
    public JsonObject handleResponse(
            final HttpResponse response) throws IOException {
        StatusLine statusLine = response.getStatusLine();
        HttpEntity entity = response.getEntity();
        if (statusLine.getStatusCode() >= 300) {
            throw new HttpResponseException(
                    statusLine.getStatusCode(),
                    statusLine.getReasonPhrase());
        }
        if (entity == null) {
            throw new ClientProtocolException("Response contains no content");
        }
        Gson gson = new GsonBuilder().create();
        ContentType contentType = ContentType.getOrDefault(entity);
        Charset charset = contentType.getCharset();
        Reader reader = new InputStreamReader(entity.getContent(), charset);
        return gson.fromJson(reader, MyJsonObject.class);
    }
};
MyJsonObject myjson = client.execute(httpget, rh);
```





## 参考文献

> [HttpClient Tutorial](http://hc.apache.org/httpcomponents-client-4.5.x/tutorial/html/index.html)
>
> [Java工具类--通过HttpClient发送http请求](https://blog.csdn.net/frankcheng5143/article/details/50070591)
>
> [木杉的博客--HttpClient学习笔记](http://imushan.com/2017/02/07/java/framework/HttpClient%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/)