---
title: JavaWeb-Tomcat
date: 2018-02-08 19:37:20
categories: 
- JavaWeb
tags:
- Tomcat
---

基础

<!--more-->

##  基础

老生常谈的双亲委托类加载机制

Tomcat 的总体结构:

![Tomcat 的总体结构](https://www.ibm.com/developerworks/cn/java/j-lo-tomcat1/image001.gif)

Connector 主要负责接收浏览器的发过来的 tcp 连接请求，创建一个 Request 和 Response 
对象分别用于和请求端交换数据，然后会产生一个线程来处理这个请求并把产生的 Request 和 Response 
对象传给处理这个请求的线程，处理这个请求的线程就是 Container 组件要做的事了。

Container 是容器的父接口，所有子容器都必须实现这个接口，分别是：Engine、Host、Context、Wrapper，四个组件前者包含后者，父子关系。通常一个 Servlet class 对应一个 Wrapper。

多个 Connector 和一个 Container 就形成了一个 Service，Service 对外提供服务，Server 提供一个接口让其它程序能够访问到这个 Service 集合、同时要维护它所包含的所有 Service 的生命周期。



context.getParameter()是获取POST/GET传递的参数值；
context.getInitParameter获取Tomcat的server.xml中设置Context的初始化参数

context.getAttribute()是获取对象容器中的数据值；
context. getRequestDispatcher是请求转发。 



热部署：[Intellij IDEA配置tomcat热部署](http://www.cnblogs.com/xuange306/p/7070938.html)



***

## 问题

### 1、新建web项目部署Tomcat的时候提示 No artifacts configured。

![dd](http://img.blog.csdn.net/20170929094556696?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd2VpeGluXzM4NTE3ODcz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

本来图示上的第二个加号点击应该会出现选择Artifact，选项有 war 或者 war exploded 。（此图为之前的正常情况下的图示）

按照网上的方法：[IntelliJ IDEA部署tomcat时Edit Configuration无artifact选项](http://blog.csdn.net/zsy3313422/article/details/52583091)

[【错误解决】Intellj（IDEA） warning no artifacts configured](http://blog.csdn.net/Small_Mouse0/article/details/77506060)

第一个方法我和评论里面一样，Web Application ：Exploded没有选项。

第二个方法，直觉不是这样，因为按照步骤来应该不会出现这种情况。

遂重建项目大法好，发现自己在建完项目忘记按右下角的Enable Auto-Import按钮……

>Import Changes选项表示会将新的依赖添加到工程中
>
>Enable Auto-Import 代表的是以后pom文件有新的变化，会自动调整工程中的相关依赖，不会每次改动都来提示了。



