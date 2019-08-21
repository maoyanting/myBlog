---
title: JavaWeb-Mybatis
date: 2018-02-09 14:11:13
categories: 
- JavaWeb
tags:
- Mybatis
---

spring-mybatis与原生mybatis使用对比、使用mybatis-generator

<!--more-->

## 使用mybatis-generator

### pom

往pom.xml添加依赖（已剔除无关的代码）

```xml
、
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.sandao</groupId>
    <artifactId>newjavaweb</artifactId>
    <packaging>war</packaging>
    <version>1.0-SNAPSHOT</version>
    <name>newjavaweb Maven Webapp</name>
    <url>http://maven.apache.org</url>

    <dependencies>
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.3.0</version>
        </dependency>
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis-spring</artifactId>
            <version>1.2.3</version>
        </dependency>
        <dependency>
            <groupId>org.mybatis.generator</groupId>
            <artifactId>mybatis-generator-core</artifactId>
            <version>1.3.4</version>
        </dependency>
    </dependencies>
    //这里注意路径
    <properties>
        <mbg.config>${basedir}/src/main/resources/generatorConfig.xml</mbg.config>
    </properties>

    <build>
        <finalName>interview_question</finalName>
        <plugins>
            <plugin>
                <groupId>org.mybatis.generator</groupId>
                <artifactId>mybatis-generator-maven-plugin</artifactId>
                <version>1.3.4</version>
                <configuration>
                    <configurationFile>${mbg.config}</configurationFile>
                </configuration>
                <dependencies>
                    <dependency>
                        <groupId>mysql</groupId>
                        <artifactId>mysql-connector-java</artifactId>
                        <version>5.1.29</version>
                    </dependency>
                </dependencies>
            </plugin>
        </plugins>
    </build>
</project>
```

mybatis和mybatis-spring是因为准备使用mybatis-spring，详见官方教程：[mybatis-spring(中文)](http://www.mybatis.org/spring/zh/index.html)和[mybatis-spring(English)](http://www.mybatis.org/spring/factorybean.html)

mybatis-generator-core 是为了自动生成Mapping的映射文件

***

### generatorConfig.xml

最详细的配置可见：[Mybatis Generator最完整配置详解](https://www.jianshu.com/p/e09d2370b796)

英文版官方配置详见：[MyBatis Generator](http://www.mybatis.org/generator/index.html)

简单配置如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>
    <context id="MySQLTables" targetRuntime="MyBatis3">
        <!--连接数据库-->
        <jdbcConnection driverClass="com.mysql.jdbc.Driver"
                        connectionURL="jdbc:mysql://localhost:3306/interview"
                        userId="root"
                        password="*******">
        </jdbcConnection>

        <javaTypeResolver>
            <property name="forceBigDecimals" value="false"/>
        </javaTypeResolver>

        <!--生成实体类 指定包名以及生成的地址（可以自定义地址，但是路径不存在不会自动创建 ） -->
        <javaModelGenerator targetPackage="com.sandao.entity" targetProject="src/main/java">
            <property name="enableSubPackages" value="true"/>
            <property name="trimStrings" value="true"/>
        </javaModelGenerator>

        <!--生成SQLMAP文件-->
        <sqlMapGenerator targetPackage="mapper" targetProject="src/main/resources">
            <property name="enableSubPackages" value="true"/>
        </sqlMapGenerator>

        <!--对应数据库表 mysql可以加入主键自增 字段命名 忽略某字段等-->
        <!--注意：把domainObjectName修改为驼峰形式-->
        <table tableName="user" domainObjectName="User"
               enableCountByExample="false"
               enableUpdateByExample="false"
               enableDeleteByExample="false"
               enableSelectByExample="false"
               selectByExampleQueryId="false">

        </table>
        <table tableName="simple_user" domainObjectName="SimpleUser"
               enableCountByExample="false"
               enableUpdateByExample="false"
               enableDeleteByExample="false"
               enableSelectByExample="false"
               selectByExampleQueryId="false">

        </table>
    </context>
</generatorConfiguration>
```



然后在路径下新建对应的dao、entity和mapper文件夹。同时在dao文件夹加新建BaseDao文件



```java
public interface BaseDao<PK, T> {

    T selectByPrimaryKey(PK id);

    int deleteByPrimaryKey(PK id);

    int insert(T record);

    int insertSelective(T record);

    int updateByPrimaryKeySelective(T record);

    int updateByPrimaryKey(T record);

    List<T> getAll();
}
```

点击IDEA右侧的Maven Project中，找到Plugins中的mybatis-generator，点击generate创建实体类和mapper文件。

DAO类大致如下：

```java
public interface UserDao extends BaseDao<Integer,User> {}
```

>这里也可以在generatorConfig.xml里面设置自动生成DAO类，但是如果需要继承BaseDao接口的时候，是不支持泛型的。
>
>且生成的接口文件名是类似于XXXMapper的，强迫症如我，需要XXXDao，所以还不如自己新建来的快。
>
>注意添加@Repository

结束。

>延伸阅读：[java web 中dao 层和service层都使用接口，是否是为使用接口而使用接口？](https://www.zhihu.com/question/36021012)



***

## 数据库操作的返回值

**insert**，返回值是：新插入行的主键（primary key）；需要包含`<selectKey>`语句，才会返回主键，否则返回值为null。
**update**/**delete**，返回值是：更新或删除的行数；无需指明resultClass；但如果有约束异常而删除失败，只能去捕捉异常。
**queryForObject**，返回的是：一个实例对象或null；需要包含`<select>`语句，并且指明resultMap；
**queryForList**，返回的是：实例对象的列表；需要包含`<select>`语句，并且指明resultMap；

***



## 动态SQL

*where* 元素只会在至少有一个子元素的条件返回 SQL 子句的情况下才去插入“WHERE”子句。而且，若语句的开头为“AND”或“OR”，*where* 元素也会将它们去除。





## 参考资料

[spring-mybatis与原生mybatis使用对比](https://juejin.im/post/5a0e9c6ff265da432528e0b0)

官方教程：[mybatis-spring(中文)](http://www.mybatis.org/spring/zh/index.html)和[mybatis-spring(English)](http://www.mybatis.org/spring/factorybean.html)