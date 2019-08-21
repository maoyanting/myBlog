---
title: Spring（未完）
date: 2018-09-17 11:56:06
categories: 
- Spring
tags:
- Spring
---

spring 是一个 **轻量级**的 实现**IOC** 和 **AOP**的框架。

<!--more-->

Spring从狭义上可以被理解成Spring Framework&SpringMVC，而广义上包含了Spring众多的开源项目。

将对象创建过程的职责赋予容器，**通过容器管理对象的生老病死**， 将对象创建过程从编译时延期到运行时，即通过配置进行加载，这样一来就解决了不用编译后期选择具体实现，其实就是面向对象的核心理念，针对接口编程。IoC开始就是个factory加上依赖管理罢了，这样一来，一个系统的创建过程就从原先的new改为配置组装，内部通过注入解决了依赖关系，只要满足接口协议即插即用。通过IoC, AOP事实上形成了一个套路，通过这个套路完成了系统的整合。

Spring常见的问题包括：Spring Bean的scope取值，BeanFactory的地位，@Transactionl相关（传播机制和隔离级别），SpringMVC工作流程

## IoC

资源不由使用资源的双方管理，而由不使用资源的第三方管理，这可以带来很多好处。

第一，资源集中管理，实现资源的可配置和易管理。

第二，降低了耦合度。

依赖注入：甲方开放接口，在它需要的时候，能够讲乙方传递进来(注入)
控制反转：甲乙双方不相互依赖，交易活动的进行不依赖于甲乙任何一方，整个活动的进行由第三方负责管理。



## AOP

参见[AOP 那点事儿](https://my.oschina.net/huangyong/blog/161338)



***

## 七大模块

1. **Spring Core**： Core封装包是框架的最基础部分，提供**IOC和依赖注入特性**。这里的基础概念是BeanFactory，它提供对Factory模式的经典实现来消除对程序性单例模式的需要，并真正地允许你从程序逻辑中分离出依赖关系和配置。
2. **Spring Context**: 构建于Core封装包基础上的 Context封装包，提供了一种框架式的对象访问方法，有些象JNDI注册器。Context封装包的特性得自于Beans封装包，并添加了对国际化（I18N）的支持（例如资源绑定），事件传播，资源装载的方式和Context的透明创建，比如说通过Servlet容器。
3. **Spring DAO**:  DAO (Data Access Object)提供了**JDBC的抽象层**，它可消除冗长的JDBC编码和解析数据库厂商特有的错误代码。 并且，JDBC封装包还提供了一种比编程性更好的声明性事务管理方法，不仅仅是实现了特定接口，而且对所有的POJOs（plain old Java objects）都适用。
4. **Spring ORM**: ORM 封装包**提供了常用的“对象/关系”映射APIs的集成层**。 其中包括JPA、JDO、Hibernate 和 iBatis 。利用ORM封装包，可以混合使用所有Spring提供的特性进行“对象/关系”映射，如前边提到的简单声明性事务管理。
5. **Spring AOP**: Spring的 AOP 封装包提供了符合AOP Alliance规范的面向方面的编程实现，让你可以定义，例如方法拦截器（method-interceptors）和切点（pointcuts），从逻辑上讲，从而减弱代码的功能耦合，清晰的被分离开。而且，利用source-level的元数据功能，还可以将各种行为信息合并到你的代码中。
6. **Spring Web**: Spring中的 Web 包提供了基础的针对Web开发的集成特性，例如多方文件上传，利用Servlet listeners进行IOC容器初始化和针对Web的ApplicationContext。当与WebWork或Struts一起使用Spring时，这个包使Spring可与其他框架结合。
7. **Spring Web MVC**: Spring中的MVC封装包提供了Web应用的Model-View-Controller（MVC）实现。Spring的MVC框架并不是仅仅提供一种传统的实现，它提供了一种清晰的分离模型，在领域模型代码和Web Form之间。并且，还可以借助Spring框架的其他特性。



## SpringCloud

最近在知乎上看了 Spring Boot 和 Dubbo 相比较的问题，很疑惑，不知道是自己的眼界过窄还是怎样，知乎上觉得多数人比较推崇 SC，对于 Dubbo 则是不推荐。当然，Dubbo 曾长期没有维护确实是让人觉得不安心，Spring Boot 在稳定性上基本是更胜一筹。

但是就我本人实际来看，SpringCloud 的各类组件用的人还是挺多的，RPC框架方面，自我感觉还是 Dubbo 更适合，对于代码的入侵性也小。

1. Feign，Ribbon，Eureka，Zuul的使用。了解各个组件的作用，会问一些常遇到的问题如Feign的重试机制，Eureka的保护机制，Zuul的路由机制等。
2. Spring Cloud使用的restful http通信与RPC通信的对比。毕竟...这是一个经久不衰的辩题，可以从耦合性，通信性能，异构系统的互信等角度对比。

#### Bean

spring可以在配置文件中配置Bean初始化函数和消亡函数

#### SpringBoot

