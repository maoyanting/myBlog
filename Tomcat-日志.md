---
title: Tomcat-日志
date: 2018-03-28 15:49:06
categories: 
- Tomcat
tags:
- Tomcat
- 日志
---

昨天装了SSH，连接到了linux的远程服务器，然后和我说可以查看Tomcat的日志

我：喵喵？

好吧我承认我没怎么搞懂日志这个东西

<!--more-->

## 零、日志的使用

首先，我们要查看日志

```shell
#查看最新300行的日志
tail -300f out.log
```

然后，我们要知道日志里面有什么

基本上，如果按照普通配置来说，你在本地运行控制台能看见的日志，基本都可以在日志文件里面看见

报错、人为打的日志，tomcat的启动日志等

报错的时候肯定是你代码有问题了，当然也可能是超时（connect timeout）什么的。

人为打的日志，基本上是代码的开发者为了标记一些状态而写的，你可以靠日志来判断代码的运行情况。







## 一、默认情况下Tomcat生成的日志文件

> catalina.out：引擎的日志文件
>
> catalina.{yyyy-MM-dd}.log：引擎的日志文件
>
> localhost.{yyyy-MM-dd}.log：Tomcat下内部代码丢出的日志
>
> host-manager.{yyyy-MM-dd}.log
>
> localhost_access_log.{yyyy-MM-dd}.txt：客户端访问日志，记录访问的url请求



## 二、日志配置文件

一般在conf/logging.properties

```properties
    #可配置项(5类日志)：catalina、localhost、manager、admin、host-manager  
    #handlers定义了可用的日志处理器名称
    handlers = 1catalina.org.apache.juli.FileHandler, 2localhost.org.apache.juli.FileHandler,  
    3manager.org.apache.juli.FileHandler, 4host-manager.org.apache.juli.FileHandler, java.util.logging.ConsoleHandler  
      
    #日志输出为输出到文件catalina.out和控制台  
    .handlers = 1catalina.org.apache.juli.FileHandler, java.util.logging.ConsoleHandler  
      
    #日志输出级别：SEVERE (最高级别) > WARNING > INFO > CONFIG > FINE > FINER(精心) > FINEST (所有内容,最低级别)
    
    #以下为各个日志处理器的配置，该日志的等级(level),日志位置(directory),日志文件的前缀(prefix)
    
    #配置文件使catalina日志输出级别为FINE  
    1catalina.org.apache.juli.FileHandler.level = FINE  
    #catalina文件输出位置  
    1catalina.org.apache.juli.FileHandler.directory = ${catalina.base}/logs  
    #catalina日志前缀为catalina  
    1catalina.org.apache.juli.FileHandler.prefix = catalina.  
      
    #配置文件使localhost日志输出级别为FINE  
    2localhost.org.apache.juli.FileHandler.level = FINE  
    #localhost文件输出位置  
    2localhost.org.apache.juli.FileHandler.directory = ${catalina.base}/logs  
    #localhost日志前缀为localhost  
    2localhost.org.apache.juli.FileHandler.prefix = localhost.  
      
    #配置文件使manager日志输出级别为FINE  
    3manager.org.apache.juli.FileHandler.level = FINE  
    #manager文件输出位置  
    3manager.org.apache.juli.FileHandler.directory = ${catalina.base}/logs  
    #manager日志前缀为manager  
    3manager.org.apache.juli.FileHandler.prefix = manager.  
      
    #配置文件使host-manager日志输出级别为FINE  
    4host-manager.org.apache.juli.FileHandler.level = FINE  
    #host-manager文件输出位置  
    4host-manager.org.apache.juli.FileHandler.directory = ${catalina.base}/logs  
    #host-manager日志前缀为host-manager  
    4host-manager.org.apache.juli.FileHandler.prefix = host-manager.  
      
    #配置文件使控制台日志输出级别为FINE  
    java.util.logging.ConsoleHandler.level = FINE  
    #控制台日志输出格式  
    java.util.logging.ConsoleHandler.formatter = java.util.logging.SimpleFormatter  
    
############################################################
# Facility specific properties.
# Provides extra control for each logger.
############################################################
      
    #localhost日志文件输出级别为INFO  
    org.apache.catalina.core.ContainerBase.[Catalina].[localhost].level = INFO  
    #localhost日志文件输出处理类2localhost.org.apache.juli.FileHandler  
    org.apache.catalina.core.ContainerBase.[Catalina].[localhost].handlers = 2localhost.org.apache.juli.FileHandler  
      
    #manager日志文件输出级别为INFO  
    org.apache.catalina.core.ContainerBase.[Catalina].[localhost].[/manager].level = INFO  
    #manager日志文件输出处理类3manager.org.apache.juli.FileHandler  
    org.apache.catalina.core.ContainerBase.[Catalina].[localhost].[/manager].handlers = 3manager.org.apache.juli.FileHandler  
      
    #host-manager日志文件输出级别为INFO  
    org.apache.catalina.core.ContainerBase.[Catalina].[localhost].[/host-manager].level = INFO  
    #host-manager日志文件输出处理类4host-manager.org.apache.juli.FileHandler  
    org.apache.catalina.core.ContainerBase.[Catalina].[localhost].[/host-manager].handlers = 4host-manager.org.apache.juli.FileHandler  
```

## 三、日志

### catalina.out

catalina.out其实是tomcat的标准输出(stdout)和标准出错(stderr)，这是在tomcat的启动脚本里指定的，如果没有修改的话stdout和stderr会重定向到这里。所以我们在应用里使用System.out打印的东西都会到这里来。

一些组件自己捕获了异常，然后将其打印了，这个时候如果恰好这些日志被我们配置成输出到console，则这些日志也会在catalina.out里出现。

另外，如果我们在应用里使用其他的日志框架，**配置了向Console输出的**，则也会在这里出现。比如以logback为例，如果配置ch.qos.logback.core.ConsoleAppender则会输出到catalina.out里。



### cataliana.{yyyy-MM-dd}.log

root输出到catalina和console。

而这里的catalina按照配置对应的是catalina.{yyyy-MM-dd}.log，这里的console最终会输出到catalina.out。这就是我们看到catalina.{yyyy-MM-dd}.log和catalina.out的日志很多都是一样的原因。

### localhost.{yyyy-MM-dd}.log

配置文件中还有一个localhost，所有logname或parent logname为org.apache.catalina.core.ContainerBase.[Catalina].[localhost]的都会输出到localhost.{yyyy-MM-dd}.log文件。而这个logname又代表着什么呢？在conf/server.xml的配置文件，其中有这么一个片段:

```xml
<Engine name="Catalina" defaultHost="localhost">
	<Host name="localhost" appBase="webapps"
		unpackWARs="false" autoDeploy="false">
	</Host>
</Engine>
```

我们可以这么简单的理解: 一个Tomcat进程对应着一个Engine，一个Engine下可以有多个Host(Virtual Host)，一个Host里可以有多个Context，比如我们常常将应用部署在ROOT下还是webapps里其他目录，这个就是Context。这其中Engine对应着tomcat里的StandardEngine类，Host对应着StandardHost类，而Context对应着StandardContext。**这几个类都是从ContainerBase派生**。

这些类里打的一些跟应用代码相关的日志都是使用ContainerBase里的getLogger，而这个这个logger的logger name就是: org.apache.catalina.core.ContainerBase.[current container name].[current container name]...

而我们一个webapp里listener, filter, servlet的初始化就是在StandardContext里进行的，比如ROOT里有一个listener初始化出异常了，打印日志则logger name是org.apache.catalina.core.ContainerBase.[Catalina].[localhost].[/]。这其中Catalina和localhost是上面xml片段里的Engine和Host的name，而[/]是ROOT对应的StandardContext的name。**所以listener, filter, servlet初始化时的日志是需要看localhost.{yyyy-MM-dd}.log这个日志的**。



### localhost_access_log.{yyyy-MM-dd}.txt

默认还会产生一个文件名格式为localhost_access_log.yyyy-MM-dd.txt的文件，**记录访问的url请求**

这个是通过conf/server.xml的`<Host>`下的`<Valve>`节点定义的，如果不需要直接注释掉`<Valve>`的定义就可以了。



## **总结**

catalina.out即标准输出和标准出错，所有输出到这两个位置的都会进入catalina.out，这里包含**tomcat运行自己输出的日志**以及**应用里向console输出的日志**。

catalina.{yyyy-MM-dd}.log是**tomcat自己运行的一些日志**，这些日志还会输出到catalina.out，但是*应用向console输出的日志不会输出到catalina.{yyyy-MM-dd}.log。*

localhost.{yyyy-MM-dd}.log主要是**应用初始化(listener, filter, servlet)未处理的异常最后被tomcat捕获而输出的日志**，而这些未处理异常最终会导致应用无法启动。

localhost_access_log.yyyy-MM-dd.txt主要记录访问的url请求。





> 参考链接：
>
> [tomcat默认的Log配置分析](http://www.javacoder.cn/?p=342)
>
> [tomcat下的日志配置详细说明](https://blog.csdn.net/xmtblog/article/details/24467469)