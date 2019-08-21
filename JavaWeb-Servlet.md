---
title: JavaWeb-Servlet
date: 2017-12-15 18:05:24
categories: 
- JavaWeb
tags:
- Servlet
---


这个只是学习笔记，我是个刚学半年编程的半吊子，里面的内容可能会出错，所以也不建议转载
我会尽力保证里面的正确性，如果有错请指出，万分感谢

<!--more-->

----------

简介：
---

**功能：**
servlet负责处理请求和发送响应的过程

**组成：**
Servlet的框架是由两个Java包组成:`javax.servlet`和`javax.servlet.http`. 
在javax.servlet包中定义了所有的Servlet类都必须实现或扩展的的通用接口和类.
在javax.servlet.http包中定义了采用**HTTP通信协议**的HttpServlet类.

----------


web.xml
-------

```
  <servlet>
    <!--咱们给servlet取个自己喜欢的名字-->
    <servlet-name>UrlServlet</servlet-name>
    <!--这里servlet的路径，自己写，用Spring的都行，比如DispatcherServlet-->
    <servlet-class>com.sandao.servlet.UrlServlet</servlet-class>
  </servlet>
  <servlet-mapping>
    <!--咱们servlet的名字，和上面的是一致的-->
    <servlet-name>UrlServlet</servlet-name>
    <!--默认匹配所有的请求，当然你也可以加一个-->
    <!--浏览器通过url找到这个，发现匹配了，嗯，我去找个名字的servlet-->
    <!--然后到上面找到servlet的路径-->
    <url-pattern>/</url-pattern>
  </servlet-mapping>
```
----------

工作流程
----

1. Web Client 向Servlet容器（Tomcat）发出Http请求

2. Servlet容器接收Web Client的请求

3. Servlet容器创建一个HttpRequest对象，将Web Client请求的信息封装到这个对象中。

4. Servlet容器创建一个HttpResponse对象

5. Servlet容器调用HttpServlet对象的service方法，把HttpRequest对象与HttpResponse对象作为参数传给 HttpServlet 对象。

6. HttpServlet调用HttpRequest对象的有关方法，获取Http请求信息。

7. HttpServlet调用HttpResponse对象的有关方法，生成响应数据。

8. Servlet容器把HttpServlet的响应结果传给Web Client。

----------

生命周期
----
**出生：**服务器启动时或者第一次请求该servlet时，就会**初始化**一个Servlet对象，也就是会执行初始化方法init

该servlet对象去处理所有客户端请求，在`service(ServletRequest req，ServletResponse res)`方法中执行

**死亡：**最后服务器关闭时，才会**销毁**这个servlet对象，执行destroy()方法。

----------
HttpServlet
-----------

----------


ServletConfig对象
----------
**获取途径：**
getServletConfig(); 

**方法：**
`getServletName();  `
获取servlet的名称，也就是我们在web.xml中配置的servlet-name

`getServletContext(); `
获取ServletContext对象，该对象的作用看下面讲解

`getInitParameter(String); `
获取在servlet中初始化参数的值。这里注意与全局初始化参数的区分。这个获取的只是在该servlet下的初始化参数

`getInitParameterNames(); `
获取在Servlet中所有初始化参数的名字，也就是key值，可以通过key值，来找到各个初始化参数的value值。注意返回的是枚举类型

```
    <init-param>
      <!--这里有个叫contextConfigLocation的参数-->
      <param-name>contextConfigLocation</param-name>
      <!--我们happy地给它赋个值-->
      <param-value>classpath*:spring/spring-*.xml</param-value>
    </init-param>
    <init-param>
      <param-name>listings</param-name>
      <param-value>true</param-value>
    </init-param>
```

----------


ServletContext对象
----------
**获取途径：**
getServletContext(); 、getServletConfig().getServletContext();　　
//其实一样的

**方法：**
tomcat为每个web项目都创建一个ServletContext实例，tomcat在启动时创建，服务器关闭时销毁，在一个web项目中共享数据，管理web项目资源，为整个web配置公共信息等，通俗点讲，就是一个web项目，就存在一个ServletContext实例，每个Servlet读可以访问到它。

1、web项目中共享数据

`setAttribute(String name, Object obj)` 
//在web项目范围内存放内容，以便让在web项目中所有的servlet读能访问到

`getAttribute(String name) `
//通过指定名称获得内容

`removeAttribute(String name) `
//通过指定名称移除内容  　

2、整个web项目初始化参数 

//这个就是全局初始化参数，每个Servlet中都能获取到该初始化值

`getInitPatameter(String name)　`　
//通过指定名称获取初始化值

`getInitParameterNames()`
//获得枚举类型

3、获取web项目资源

`getServletContext().getRealPath("/WEB-INF/web.xml")`
获取web项目下指定资源的路径

`InputStream getResourceAsStream(java.lang.String path)`
获取web项目下指定资源的内容，返回的是字节输入流。

4、`getResourcePaths(java.lang.String path) ` 指定路径下的所有内容。

----------

request对象
---------
简介：request就是将请求文本封装而成的对象

**方法：**

1、请求行的获取

2、请求头的获取

3、请求体的获取
`String request.getParameter(String) `
获得指定名称，一个请求参数值。
//这个应该用的比较多吧

`String[] request.getParameterValues(String) `
获得指定名称，所有请求参数值。例如：checkbox、select等

`Map<String , String[]> request.getParameterMap() `
获得所有的请求参数　

4、请求转发

```
request.getRequestDispatcher(String path).forward(request,response);　
```

//path:转发后跳转的页面，这里不管用不用"/"开头，都是以web项目根开始，因为这是请求转发，请求转发只局限与在同一个web项目下使用，所以这里一直都是从web项目根下开始的，

web项目根：
开发：G:\Workspaces\test01\WebRoot\..
运行时：D:\java\tomcat\apache-tomcat-7.0.53\webapps\test01\..

web站点根：
运行时：D:\java\tomcat\apache-tomcat-7.0.53\webapps\..

从这里可以看出，web项目根就是从该web项目名开始，所以我们请求转发时，只需要接着项目名后面需要访问的路径写就行了，

特点：浏览器中url不会改变，也就是浏览器不知道服务器做了什么，是服务器帮我们跳转页面的，并且在转发后的页面，能够继续使用原先的request，因为是原先的request，所以request域中的属性都可以继续获取到。

//这里有个问题
这里应该是浏览器的请求进来了，Tomcat把resquest给了servlet，servlet到手一看，emmm，你这个url应该转去这个path呢（这里的path过去的是什么文件？html？）
去吧，我把request和responce也给你一起带过去
　　　　　　　　　　　　　　　　
转发和重定向的区别：http://blog.csdn.net/u011794238/article/details/46801761

----------

response对象
----------
**方法：**
`response.setHeader(java.lang.String name, java.lang.String value) `
设置指定的头，一般常用。

重定向(页面跳转)

方式一：手动方案

```
response.setStatus(302);　　//状态码302就代表重定向
response.setHeader("location","http://www.baidu.com");
```

方式二：使用封装好的，通过`response.sendRedirect("http://www.baidu.com");`

特点：服务器告诉浏览器要跳转的页面，是浏览器主动去跳转的页面，浏览器知道，浏览器的地址栏中url会变，**是浏览器重新发起一个请求到另外一个页面**，所以request是重新发起的，跟请求转发不一样。
//这个我觉得大概使用就是你登录后跳转到别的页面的后端实现

第一种：response.sendRedirect("/test01/MyServlet01");　　
//使用了"/"开头，说明是从web站点根开始

第二种：response.sendRedirect("MyServlet01");　　
//没有使用"/"开头，说明是从web项目根开始

重定向没有任何局限，可以重定向web项目内的任何路径，也可以访问别的web项目中的路径，并且这里就用"/"区分开来，如果使用了"/"开头，就说明我要重新开始定位了，不访问刚才的web项目，自己写项目名，如果没有使用"/"开始，那么就知道是访问刚才那个web项目下的servlet，就可以省略项目名了。就是这样来区别。

----------

DispatcherServlet
-----------------

DispatcherServlet很熟悉了吧，具体不展开了

大概流程如下：
![这里写图片描述](http://img.blog.csdn.net/20171215132740885?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd2VpeGluXzM4NTE3ODcz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
文字简写版：
1、对Http请求进行初步处理，查找与之对应的Controller处理类（方法）—— HandlerMapping
2、调用相应的Controller处理类（方法）完成业务逻辑——HandlerAdapter
3、对Controller处理类（方法）调用时可能发生的异常进行处理——HandlerExceptionResolver
4、根据Controller处理类（方法）的调用结果，进行Http响应处理——ViewResolver

其实DispatcherServlet最终继承的是HttpServlet，所以行为规范和servlet是一样的
![这里写图片描述](http://img.blog.csdn.net/20171215152119301?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd2VpeGluXzM4NTE3ODcz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

----------

Tomcat
------
**工作原理部分：**
![Tomcat 的总体结构](http://img.blog.csdn.net/20171215144922607?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd2VpeGluXzM4NTE3ODcz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

从上图可以看出来：
一个 Container 可以选择对应多个 Connector。
多个 Connector 和一个 Container 就形成了一个 Service。
一个Service只能有一个Container容器。

**Connector（多个） **
主要任务：负责接收浏览器的发过来的 tcp 连接请求，创建一个 Request 和 Response 对象分别用于和请求端交换数据，然后会产生一个线程来处理这个请求，并把产生的 Request 和 Response 对象传给处理这个请求的线程，处理这个请求的线程就是 Container 组件要做的事了。

**Container（单个）**
Container 是容器的父接口，所有子容器都必须实现这个接口，Container 容器的设计用的是典型的责任链的设计模式
它有四个子容器组件构成，分别是：Engine、Host、Context、Wrapper
这四个组件不是平行的，而是父子关系，
Engine 包含 Host,Host 包含 Context，Context 包含 Wrapper。
Engine 《Host《Context《 Wrapper。
通常一个 Servlet class 对应一个 Wrapper，如果有多个 Servlet 就可以定义多个 Wrapper，如果有多个 Wrapper 就要定义一个更高的 Container 了

//这里推荐看许令波的《深入分析Java Web设计模式》
https://www.ibm.com/developerworks/cn/java/j-lo-tomcat1/index.html

**工作流程：**

1、用户点击网页内容，请求被发送到本机端口8080，被在那里监听的Coyote HTTP/1.1 Connector获得。 

2、Connector把该请求交给它所在的Service的Engine来处理，并等待Engine的回应。 

3、Engine获得请求localhost/test/index.jsp，匹配所有的虚拟主机Host。 

4、Engine匹配到名为localhost的Host（即使匹配不到也把请求交给该Host处理，因为该Host被定义为该Engine的默认主机），名为localhost的Host获得请求/test/index.jsp，匹配它所拥有的所有的Context。Host匹配到路径为/test的Context（如果匹配不到就把该请求交给路径名为“ ”的Context去处理）。 

5、path=“/test”的Context获得请求/index.jsp，在它的mapping table中寻找出对应的Servlet。Context匹配到URL PATTERN为*.jsp的Servlet,对应于JspServlet类。 

6、构造HttpServletRequest对象和HttpServletResponse对象，作为参数调用JspServlet的doGet（）或doPost（）.执行业务逻辑、数据存储等程序。 

7、Context把执行完之后的HttpServletResponse对象返回给Host。 

8、Host把HttpServletResponse对象返回给Engine。 

9、Engine把HttpServletResponse对象返回Connector。 

10、Connector把HttpServletResponse对象返回给客户Browser

----------

参考资料：
-----

Java Web(一) Servlet详解！！http://www.cnblogs.com/whgk/p/6399262.html
servlet 运行工作原理详解  http://www.51gjie.com/javaweb/847.html
Spring MVC中DispatcherServlet工作原理探究  http://blog.csdn.net/hzzhoushaoyu/article/details/6853730
SpringMVC深度探险：http://downpour.iteye.com/category/196182
//必看，作者写的超级好，可惜去炒股了就不写blog了