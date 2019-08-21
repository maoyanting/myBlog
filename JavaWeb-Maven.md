---
title: JavaWeb-Maven
date: 2018-02-08 20:09:53
categories: 
- JavaWeb
tags:
- Maven
---

依赖寻找和设置，POM，SSM必要依赖等

<!--more-->

## 设置

### 查找依赖包

推荐[sonatype RSO](https://repository.sonatype.org/index.html#nexus-search;quick~mybatis-generator-maven-plugin)

就是有时候有点慢

### 设置中央仓库的镜像

maven默认的 central 中央仓库，指向的地址是 `repo.maven.apache.org` 。

这个地址经常出现问题，比如不能访问，或者速度超慢（低于1k/s）。因此在国内最好找一个国内的镜像站点来避免直接访问 central 中央仓库。

参考：[设置中央仓库的镜像](https://skyao.gitbooks.io/learning-maven/content/installation/mirror.html)

***

## POM

[@Maven POM 详解](https://www.jianshu.com/p/8417a94c4d94)

最基本的pom.xml如下：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>//POM 模型版本号 (通常为 4.0.0)
    <groupId>com.sandao</groupId>//组织或者公司名，通常是反向域名
    <artifactId>newjavaweb</artifactId>
    <packaging>war</packaging>//项目的包装方式，可选，默认为jar或者war
    <version>1.0-SNAPSHOT</version>//当前版本号
    <name>newjavaweb Maven Webapp</name>
    <url>http://maven.apache.org</url>
    
 <dependencies>
    <dependency>
      <groupId>junit</groupId>//这个依赖所属于的组织或公司
      <artifactId>junit</artifactId>//我们需要的依赖的程序库
      <version>4.12</version>//版本号
      <scope>test</scope>//作用域
    </dependency>
 <dependencies>  
    <build>
        <finalName>interview_question</finalName>
    </build>
</project>
```

>延伸资料：[tar包和jar包和war包的区别？](https://www.zhihu.com/question/22129866)
>

### SSM必要的依赖

>这里是我觉得必要的依赖，网上没找到讲这方面的，如有错请务必提出来。

| 依赖名                  | 功能                                                         |
| :---------------------- | ------------------------------------------------------------ |
| **———测试—————**        |                                                              |
| junit                   |                                                              |
| **———日志—————**        |                                                              |
| slf4j-api               | 使用lSlf4j+logback组合                                       |
| logback-core            | 其余组合详见[日志工具现状调研](http://tech.lede.com/2017/02/06/rd/server/log4jSearch/)、[混乱的 Java 日志体系](http://www.ixirong.com/2016/03/13/intro-to-java-log/) |
| logback-classic(集成包) |                                                              |
| **———数据库—————**      |                                                              |
| mysql-connector-java    | 使用了c3p0，其余选择见[c3p0数据库连接池介绍+实例](http://blog.csdn.net/u012050416/article/details/50738892) |
| c3p0                    |                                                              |
| **———Mybatis—————**     |                                                              |
| mybatis                 | 使用了mybatis-spring                                         |
| mybatis-spring          | 具体使用详见[mybatis-spring(中文)](http://www.mybatis.org/spring/zh/index.html)和[mybatis-spring(English)](http://www.mybatis.org/spring/factorybean.html) |
| mybatis-generator-core  | 辅助：自动生成Mapping的映射文件                              |
| **———Spring—————**      |                                                              |
| spring-webmvc           |                                                              |
| spring-test             | 配合junit使用，详见[[Spring Test 整合 JUnit 4 使用总结](http://www.cnblogs.com/rainisic/archive/2012/01/22/Spring_Test_Framework.html)] |
| spring-tx               | spring提供对事务的支持                                       |
|                         | 其余可参考[Spring3.2.2中相关Jar包的作用](http://www.cnblogs.com/zhangminghui/p/3455782.html) |
| **———Json—————**        |                                                              |
| fastjson                | ali的一个开源json软件                                        |
|                         |                                                              |

这里重点提一下spring-webmvc，这里导入的是4.3.13.RELEASE，同时，还会隐式导入

- spring-beans:

- spring-context:

- spring-aop:

- spring-core:

- spring-web:

- spring-expression:

- commons-logging:

- spring-context-support

- spring-oxm

- javax.servlet-api

  等依赖，所以，如果你导入了spring-webmvc，这些依赖就不需要重复写一遍了。