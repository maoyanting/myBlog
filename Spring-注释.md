---
title: Spring-注释
date: 2018-01-11 15:49:06
categories: 
- Spring
tags:
- Spring
---



## 注释

#### @Component：

功能：表明该类会作为组件类，并告知Spring要为此创建bean

下述这 3 个注释和 @Component 是等效的，但是从注释类的命名上，很容易看出这 3 个注释分别和持久层、业务层和控制层（Web 层）相对应。

##### @Service

用于标注业务层组件

##### @Controller

用于标注控制层组件

##### @Repository

用于标注数据访问组件，即DAO组件

##### @Component

泛指组件，当组件不好归类的时候，我们可以使用这个注解进行标注。

***

#### @ComponentScan：

功能：在Spring中启用组件扫描（默认组件扫描不启动），默认扫描与配置类相同的包；

@ComponentScan（”packageName"）

指明是基础包：@ComponentScan（basePackages=”package1Name”）@ComponentScan（basePackages={”package1Name”,”package2Name”}）

另一种方式：@ComponentScan（basePackagesClasses={ package1Name.class , package2Name.class } ）

***

#### @Autowired:

功能：声明要进行自动装配

使用：在构造器和方法 上

默认**按类型装配**，默认情况下必须要求依赖对象必须存在，如果要允许null值，可以设置它的required属性为false，如：@Autowired(required=false) ，如果我们想使用名称装配可以结合@Qualifier注解进行使用。

加上@Autowired后不需要getter()和setter()方法，Spring也会自动注入。   

***

#### @Qualifier

功能：按名字装配bean

当我们在 Spring 容器中配置了两个类型为 Office 类型的 Bean，当对 Boss 的 office 成员变量进行自动注入时，Spring 容器将无法确定到底要用哪一个 Bean，就会发生异常。

Spring 允许我们通过 @Qualifier 注释指定注入 Bean 的名称，这样歧义就消除了，所以 @Autowired 和 @Qualifier 结合使用时，自动注入的策略就从 byType 转变成 byName 了。

***

#### @Resource

功能：按名字装配bean

是**JDK1.6支持的注解**（非Spring），默认按照名称进行装配。

名称可以通过name属性进行指定，如果没有指定name属性，当注解写在字段上时，默认取字段名，按照名称查找，如果注解写在setter方法上默认取属性名进行装配。当找不到与名称匹配的bean时才按照类型进行装配。但是需要注意的是，如果name属性一旦指定，就只会按照名称进行装配。

***

#### @Configuration：

表明此类是个配置类

***

#### @Bean：

功能：声明简单的bean

这个方法将会返回一个对象并注册为Spring应用上下文中的bean。方法体中包含了最终产生bean实例的逻辑；

@Bean（name:”beanName”），默认为方法名

@Pointcut

定义命名的切点

@Pointcut：（“execution(* 全限定类名  .  方法  (..))”）

public void performance（）{}

后续可以直接使用切点标示：

@Before（“ performance（）”）

***

#### @EnableWebMvc

功能：启动SpringMVC

***

#### @RequestMapping

功能：通过匹配URL路径来访问相应页面的

分类：类级别的和方法级别的

***

## 参考资料

Spring注释详解：[http://blog.csdn.net/xyh820/article/details/7303330](http://blog.csdn.net/xyh820/article/details/7303330)

[http://blog.csdn.net/zhang854429783/article/details/6785574](http://blog.csdn.net/zhang854429783/article/details/6785574)