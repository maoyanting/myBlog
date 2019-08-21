---
title: 翻译-Building a RESTful Web Service
date: 2018-02-14 21:00:27
categories: 
- 翻译
tags:
- RESTful
- Spring
---

翻译——使用Spring创建一个RESTful Web Service

<!--more-->

# 创建一个RESTful Web Service

 本指南将引导您完成使用Spring创建“hello world” [RESTful Web服务的过程](https://translate.googleusercontent.com/translate_c?depth=1&hl=zh-CN&rurl=translate.google.com&sl=auto&sp=nmt4&tl=zh-CN&u=https://spring.io/understanding/REST&usg=ALkJrhjA4okemhma2AiF2YSmzUKeGHZEsg) 。 

##  你会建立什么 

 您将在以下位置创建一个接受HTTP GET请求的服务： 

```
http://localhost:8080/greeting
```

 并用一个[JSON](https://translate.googleusercontent.com/translate_c?depth=1&hl=zh-CN&rurl=translate.google.com&sl=auto&sp=nmt4&tl=zh-CN&u=https://spring.io/understanding/JSON&usg=ALkJrhiAdpYHVloWmoMLbhBu221YaBxk1w)表示欢迎： 

```
 {"id":1,"content":"Hello, World!"} 
```

 您可以使用查询字符串中可选的`name`参数来自定义问候语： 

```
http://localhost:8080/greeting?name=User
```

  `name`参数值覆盖“World”的默认值，并反映在response中： 

```
 {"id":1,"content":"Hello, User!"} 
```

##  你需要什么 

-  大约15分钟 
-  喜欢的文本编辑器或IDE 
-   [JDK 1.8](https://translate.googleusercontent.com/translate_c?depth=1&hl=zh-CN&rurl=translate.google.com&sl=auto&sp=nmt4&tl=zh-CN&u=http://www.oracle.com/technetwork/java/javase/downloads/index.html&usg=ALkJrhi2isGPHV-sq7jUziPVE-l9jBalvg)或更高版本 
-   [Gradle 2.3+](https://translate.googleusercontent.com/translate_c?depth=1&hl=zh-CN&rurl=translate.google.com&sl=auto&sp=nmt4&tl=zh-CN&u=http://www.gradle.org/downloads&usg=ALkJrhjd_PiqFxW7gUhy0tzUXfe3aZH-1A)或[Maven 3.0+](https://translate.googleusercontent.com/translate_c?depth=1&hl=zh-CN&rurl=translate.google.com&sl=auto&sp=nmt4&tl=zh-CN&u=https://maven.apache.org/download.cgi&usg=ALkJrhj_AbU4UixGaWViIk58fBiQPaEg9A) 
-  您还可以直接将代码导入到您的IDE中： 
  -   [Spring工具套件（STS）](https://translate.googleusercontent.com/translate_c?depth=1&hl=zh-CN&rurl=translate.google.com&sl=auto&sp=nmt4&tl=zh-CN&u=https://spring.io/guides/gs/sts&usg=ALkJrhiKfyyZVm8Ccgl2yvAU_rBxEYHlIA) 
  -   [IntelliJ IDEA](https://translate.googleusercontent.com/translate_c?depth=1&hl=zh-CN&rurl=translate.google.com&sl=auto&sp=nmt4&tl=zh-CN&u=https://spring.io/guides/gs/intellij-idea/&usg=ALkJrhhcC0vJb4UeMWHkEiTZjFgFEM065Q) 

##  如何完成本指南 

像大多数Spring [入门指南](https://translate.googleusercontent.com/translate_c?depth=1&hl=zh-CN&rurl=translate.google.com&sl=auto&sp=nmt4&tl=zh-CN&u=https://spring.io/guides&usg=ALkJrhgRlfg-cgquXd16m2zYJRELyIHD8A)一样，您可以从头开始并完成每个步骤，也可以绕过已熟悉的基本设置步骤。 无论哪种方式，你最终都能得到成功的代码。 

要**从头开始** ，请继续阅读[使用Gradle构建](https://translate.googleusercontent.com/translate_c?depth=1&hl=zh-CN&rurl=translate.google.com&sl=auto&sp=nmt4&tl=zh-CN&u=https://spring.io/guides/gs/rest-service/&usg=ALkJrhh5TggyMiWSsyF0qIYKW_Py__4VYw#scratch) 。 

要**跳过这些基础知识** ，请执行以下操作： 

-   [下载](https://github.com/spring-guides/gs-rest-service/archive/master.zip)并解压缩本指南的源代码库，或使用[Git](https://translate.googleusercontent.com/translate_c?depth=1&hl=zh-CN&rurl=translate.google.com&sl=auto&sp=nmt4&tl=zh-CN&u=https://spring.io/understanding/Git&usg=ALkJrhiGT-aa_xL0VmXxdVXwTGjValsEkA)克隆它： `git clone https://github.com/spring-guides/gs-rest-service.git` 
-   cd进入`gs-rest-service/initial` 
-  跳转到[创建资源表示类](https://translate.googleusercontent.com/translate_c?depth=1&hl=zh-CN&rurl=translate.google.com&sl=auto&sp=nmt4&tl=zh-CN&u=https://spring.io/guides/gs/rest-service/&usg=ALkJrhh5TggyMiWSsyF0qIYKW_Py__4VYw#initial) 。 

  **完成后** ，您可以根据`gs-rest-service/complete`中的代码检查结果。 

##  用Gradle构建 

##  用Maven构建 

##  用您的IDE构建 

##  创建一个资源表示类 

 现在您已经设置了项目和构建系统，您可以创建您的Web服务。 

 考虑服务交互开始流程。 

该服务将处理`/greeting`的 `GET`请求，可选择性地在查询字符串中使用`name`参数。  `GET`请求应该在表示问候的正文中返回带有JSON的`200 OK`响应。 它应该如下所示： 

```
 { "id": 1, "content": "Hello, World!" } 
```

 `id`字段是问候语的唯一标识符， `content`是问候语的文字表示。 

为了对问候表示进行建模，您可以创建一个资源表示类。 为`id`和`content`数据提供一个普通的java对象，其中包含字段，构造函数和访问器： 

 `src/main/java/hello/Greeting.java` 

```java
package hello;

public class Greeting {

    private final long id;
    private final String content;

    public Greeting(long id, String content) {
        this.id = id;
        this.content = content;
    }

    public long getId() {
        return id;
    }

    public String getContent() {
        return content;
    }
}
```

>正如您在下面的步骤中看到的，Spring使用[Jackson JSON](https://translate.googleusercontent.com/translate_c?depth=1&hl=zh-CN&rurl=translate.google.com&sl=auto&sp=nmt4&tl=zh-CN&u=http://wiki.fasterxml.com/JacksonHome&usg=ALkJrhj-SRgPHPGJwDCVc6iKW6RJ6EkmFA)库自动将类型为`Greeting`实例编组为JSON。 

 接下来，您将创建将提供这些问候的资源控制器。 

##  创建一个资源控制器 

 在Spring构建RESTful Web服务的方法中，HTTP请求由控制器处理。 这些组件可以通过[`@RestController`](https://translate.googleusercontent.com/translate_c?depth=1&hl=zh-CN&rurl=translate.google.com&sl=auto&sp=nmt4&tl=zh-CN&u=https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/bind/annotation/RestController.html&usg=ALkJrhj2jREdSv48jCAFQ8mkmMNuTdphDQ)批注轻松识别，并且`GreetingController`通过返回`Greeting`类的新实例来处理`/greeting`的`GET`请求： 

 `src/main/java/hello/GreetingController.java` 

```java
package hello;

import java.util.concurrent.atomic.AtomicLong;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class GreetingController {

    private static final String template = "Hello, %s!";
    private final AtomicLong counter = new AtomicLong();

    @RequestMapping("/greeting")
    public Greeting greeting(@RequestParam(value="name", defaultValue="World") String name) {
        return new Greeting(counter.incrementAndGet(),
                            String.format(template, name));
    }
}
```

 这个控制器简洁明了，但有很多内容。 让我们一步一步分解它。 

  `@RequestMapping`注释可确保对`/greeting` HTTP请求映射到`greeting()`方法。 



>上面的例子没有指定`GET`与`PUT` ， `POST`等等，因为`@RequestMapping`默认映射所有的HTTP操作。 使用`@RequestMapping(method=GET)`来缩小这个映射。 



 `@RequestParam`将查询字符串参数`name`的值绑定到`greeting()`方法的`name`参数中。 此查询字符串参数显式标记为可选（默认情况下为`required=true` ）：如果请求中没有带name参数，则使用`defaultValue` “World”。 

方法体的实现基于`counter`的下一个值创建并返回一个带有`id`和`content`属性的新`Greeting`对象，并使用问候`template`格式化给定`name` 。 

传统的MVC控制器和上面的RESTful Web服务控制器之间一个主要区别在于HTTP响应主体的创建方式。 这个RESTful Web服务控制器只需填充并返回一个`Greeting`对象，而不是依赖[视图技术](https://translate.googleusercontent.com/translate_c?depth=1&hl=zh-CN&rurl=translate.google.com&sl=auto&sp=nmt4&tl=zh-CN&u=https://spring.io/understanding/view-templates&usg=ALkJrhiiKud_Qn3fmLH4xFp3bRaUhkjz2A)将问候数据的服务器端呈现呈现给HTML。 对象数据将作为JSON直接写入HTTP响应。 

此代码使用Spring 4的新的[`@RestController`](https://translate.googleusercontent.com/translate_c?depth=1&hl=zh-CN&rurl=translate.google.com&sl=auto&sp=nmt4&tl=zh-CN&u=https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/bind/annotation/RestController.html&usg=ALkJrhj2jREdSv48jCAFQ8mkmMNuTdphDQ)注释，该注释将类标记为控制器，其中每个方法都返回一个域对象而不是视图。 它是`@Controller`和`@ResponseBody`的缩写。 

 `Greeting`对象必须转换为JSON。 由于Spring的HTTP消息转换器支持，您不需要手动执行此转换。 由于[Jackson 2](https://translate.googleusercontent.com/translate_c?depth=1&hl=zh-CN&rurl=translate.google.com&sl=auto&sp=nmt4&tl=zh-CN&u=http://wiki.fasterxml.com/JacksonHome&usg=ALkJrhj-SRgPHPGJwDCVc6iKW6RJ6EkmFA)位于类路径中， [`MappingJackson2HttpMessageConverter`](https://translate.googleusercontent.com/translate_c?depth=1&hl=zh-CN&rurl=translate.google.com&sl=auto&sp=nmt4&tl=zh-CN&u=https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/http/converter/json/MappingJackson2HttpMessageConverter.html&usg=ALkJrhjRG0KiqvFIa-ROG_uk7sOLBnwMWw)会自动选择Spring的[`MappingJackson2HttpMessageConverter`](https://translate.googleusercontent.com/translate_c?depth=1&hl=zh-CN&rurl=translate.google.com&sl=auto&sp=nmt4&tl=zh-CN&u=https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/http/converter/json/MappingJackson2HttpMessageConverter.html&usg=ALkJrhjRG0KiqvFIa-ROG_uk7sOLBnwMWw)将`Greeting`实例转换为JSON。 

## 使应用程序可执行 

虽然可以将此服务作为传统[WAR](https://translate.googleusercontent.com/translate_c?depth=1&hl=zh-CN&rurl=translate.google.com&sl=auto&sp=nmt4&tl=zh-CN&u=https://spring.io/understanding/WAR&usg=ALkJrhjIuNhNJ-rDfCCslAH8c2nm7VJLOQ)文件打包以部署到外部应用程序服务器，但下面演示的更简单的方法会创建独立应用程序。 您将所有内容打包到一个单独的、可执行的JAR文件中，由一个Java `main()`方法驱动。 过程中，您使用Spring的支持将[Tomcat](https://translate.googleusercontent.com/translate_c?depth=1&hl=zh-CN&rurl=translate.google.com&sl=auto&sp=nmt4&tl=zh-CN&u=https://spring.io/understanding/Tomcat&usg=ALkJrhj3Th8AEUCyP-Wu97HP4kFNHf2KCQ) servlet容器作为HTTP运行时嵌入，而不是部署到外部实例。 

 `src/main/java/hello/Application.java` 

```java
package hello;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

  `@SpringBootApplication`是一个方便的注释，它增加了以下所有内容： 

-   `@Configuration`将类标记为应用程序上下文的bean定义的来源。 
-   `@EnableAutoConfiguration`通知Spring Boot根据类路径设置，其他bean和各种属性设置开始添加bean。 
-  通常你会为Spring MVC应用程序添加`@EnableWebMvc` ，但Spring Boot在类路径中看到**spring-webmvc**时会自动添加。 这将该应用程序标记为Web应用程序并激活关键行为，例如设置`DispatcherServlet` 。 
-   `@ComponentScan`告诉Spring在`hello`包中查找其他组件，配置和服务，以便找到控制器。 

`main()`方法使用Spring Boot的`SpringApplication.run()`方法启动应用程序。 你有没有注意到没有一行XML？ 没有**web.xml**文件。 这个Web应用程序是100％纯Java，您不必处理配置任何管道或基础设施。 

### 构建一个可执行的JAR 

您可以使用Gradle或Maven从命令行运行应用程序。 或者您可以构建一个包含所有必需的依赖项，类和资源的可执行JAR文件，并运行该文件。 这使得在整个开发生命周期内跨越不同环境等，将服务作为应用程序发布，版本化和部署变得非常容易。 

如果您正在使用Gradle，则可以使用`./gradlew bootRun`运行该应用程序。 或者您可以使用`./gradlew build` JAR文件。 然后你可以运行JAR文件： 

```
  java -jar build / libs / gs-rest-service-0.1.0.jar 
```

 如果您使用的是Maven，则可以使用`./mvnw spring-boot:run`来运行该应用程序。 或者，您可以使用`./mvnw clean package`构建JAR文件。 然后你可以运行JAR文件： 

```
  java -jar target / gs-rest-service-0.1.0.jar 
```

> 上述过程将创建一个可运行的JAR。 您也可以选择[构建经典的WAR文件](https://translate.googleusercontent.com/translate_c?depth=1&hl=zh-CN&rurl=translate.google.com&sl=auto&sp=nmt4&tl=zh-CN&u=https://spring.io/guides/gs/convert-jar-to-war/&usg=ALkJrhg_q399dCbdi_hrE92GWpqBBT0Zvw) 。 

 记录输出显示。 该服务应该在几秒钟内启动并运行。 

##  测试服务 

 现在服务已启动，请访问[http://localhost:8080/greeting](http://localhost:8080/greeting) ，您会看到： 

```
 {"id":1,"content":"Hello, World!"} 
```

使用[http://localhost:8080/greeting?name=User](http://localhost:8080/greeting?name=User)，提供`name`的字符串参数。 请注意`content`属性的值是如何从“Hello，World！”更改的 到“Hello User！”： 

```
 {"id":2,"content":"Hello, User!"} 
```

此更改演示了`GreetingController`中的`@RequestParam`按预期工作。  `name`参数已被赋予默认值“World”，但始终可以通过传入字符串显式覆盖。 

还要注意`id`属性如何从`1`更改为`2` 。 这证明您正在使用相同`GreetingController`实例处理多个请求，并且按照预期，在每次调用中其`counter`字段正在递增。 

##  总结 

 恭喜！ 您刚刚用Spring开发了一个RESTful Web服务。 

##  也可以看看 

 以下指南也可能有所帮助： 

-   [使用REST访问GemFire数据](https://translate.googleusercontent.com/translate_c?depth=1&hl=zh-CN&rurl=translate.google.com&sl=auto&sp=nmt4&tl=zh-CN&u=https://spring.io/guides/gs/accessing-gemfire-data-rest/&usg=ALkJrhj1OmuMq7kjSpX7uShbVE-zxLwx_Q) 
-   [使用REST访问MongoDB数据](https://translate.googleusercontent.com/translate_c?depth=1&hl=zh-CN&rurl=translate.google.com&sl=auto&sp=nmt4&tl=zh-CN&u=https://spring.io/guides/gs/accessing-mongodb-data-rest/&usg=ALkJrhgK7im05koFc2NpxrldC_xEgoC39w) 
-   [使用MySQL访问数据](https://translate.googleusercontent.com/translate_c?depth=1&hl=zh-CN&rurl=translate.google.com&sl=auto&sp=nmt4&tl=zh-CN&u=https://spring.io/guides/gs/accessing-data-mysql/&usg=ALkJrhgxcX3RIzyxbdBLJZ-zK4H27_fKVg) 
-   [使用REST访问JPA数据](https://translate.googleusercontent.com/translate_c?depth=1&hl=zh-CN&rurl=translate.google.com&sl=auto&sp=nmt4&tl=zh-CN&u=https://spring.io/guides/gs/accessing-data-rest/&usg=ALkJrhhYAb-t7bTTQo0y2bOIcSuTMqyGiA) 
-   [使用REST访问Neo4j数据](https://translate.googleusercontent.com/translate_c?depth=1&hl=zh-CN&rurl=translate.google.com&sl=auto&sp=nmt4&tl=zh-CN&u=https://spring.io/guides/gs/accessing-neo4j-data-rest/&usg=ALkJrhgJ2STjGhA8Cz5dDgpA9mN0whXMvA) 
-   [使用RESTful Web服务](https://translate.googleusercontent.com/translate_c?depth=1&hl=zh-CN&rurl=translate.google.com&sl=auto&sp=nmt4&tl=zh-CN&u=https://spring.io/guides/gs/consuming-rest/&usg=ALkJrhjXaFKj82jVDmdmEAssGF5vQpLyEg) 
-   [在AngularJS中使用REST风格的Web服务](https://translate.googleusercontent.com/translate_c?depth=1&hl=zh-CN&rurl=translate.google.com&sl=auto&sp=nmt4&tl=zh-CN&u=https://spring.io/guides/gs/consuming-rest-angularjs/&usg=ALkJrhjSBHfooUAXrstjhIMdybuu4e-w9g) 
-   [使用jQuery来使用RESTful Web服务](https://translate.googleusercontent.com/translate_c?depth=1&hl=zh-CN&rurl=translate.google.com&sl=auto&sp=nmt4&tl=zh-CN&u=https://spring.io/guides/gs/consuming-rest-jquery/&usg=ALkJrhgS_ZOF9YypRaZZ6xD-vbnhfi0SuQ) 
-   [与rest.js一起使用RESTful Web服务](https://translate.googleusercontent.com/translate_c?depth=1&hl=zh-CN&rurl=translate.google.com&sl=auto&sp=nmt4&tl=zh-CN&u=https://spring.io/guides/gs/consuming-rest-restjs/&usg=ALkJrhgjrvSyKWtPjgAltOCWlPudTAHVhg) 
-   [保护Web应用程序](https://translate.googleusercontent.com/translate_c?depth=1&hl=zh-CN&rurl=translate.google.com&sl=auto&sp=nmt4&tl=zh-CN&u=https://spring.io/guides/gs/securing-web/&usg=ALkJrhgf6Pj_Jfm7_K0lofEyyXuOs9zY_A) 
-   [用Spring构建REST服务](https://translate.googleusercontent.com/translate_c?depth=1&hl=zh-CN&rurl=translate.google.com&sl=auto&sp=nmt4&tl=zh-CN&u=https://spring.io/guides/tutorials/bookmarks/&usg=ALkJrhjS0qWCIYxfR8XHb16LKeT9Ar9oMA) 
-   [React.js和Spring Data REST](https://translate.googleusercontent.com/translate_c?depth=1&hl=zh-CN&rurl=translate.google.com&sl=auto&sp=nmt4&tl=zh-CN&u=https://spring.io/guides/tutorials/react-and-spring-data-rest/&usg=ALkJrhinRUyarQ1ByM_ZytJrx-ge79HdKQ) 
-   [使用Spring Boot构建应用程序](https://translate.googleusercontent.com/translate_c?depth=1&hl=zh-CN&rurl=translate.google.com&sl=auto&sp=nmt4&tl=zh-CN&u=https://spring.io/guides/gs/spring-boot/&usg=ALkJrhiKg3GyTsXISfFB9fKLMRE_j27MBA) 
-   [使用Restdocs创建API文档](https://translate.googleusercontent.com/translate_c?depth=1&hl=zh-CN&rurl=translate.google.com&sl=auto&sp=nmt4&tl=zh-CN&u=https://spring.io/guides/gs/testing-restdocs/&usg=ALkJrhhQ2pTFcypwxGRCdCD0pI3VSVvHGA) 
-   [启用RESTful Web服务的跨源请求](https://translate.googleusercontent.com/translate_c?depth=1&hl=zh-CN&rurl=translate.google.com&sl=auto&sp=nmt4&tl=zh-CN&u=https://spring.io/guides/gs/rest-service-cors/&usg=ALkJrhhOC4B1tMgfnh4XKkiKp6b4B8VFvA) 
-   [构建超媒体驱动的RESTful Web服务](https://translate.googleusercontent.com/translate_c?depth=1&hl=zh-CN&rurl=translate.google.com&sl=auto&sp=nmt4&tl=zh-CN&u=https://spring.io/guides/gs/rest-hateoas/&usg=ALkJrhjjhbkd7UX2EPn7FaBHS6P-Ves1oA) 
-   [断路器](https://translate.googleusercontent.com/translate_c?depth=1&hl=zh-CN&rurl=translate.google.com&sl=auto&sp=nmt4&tl=zh-CN&u=https://spring.io/guides/gs/circuit-breaker/&usg=ALkJrhgTFRUyx7gpbEH8VgTH8hCgKXeYsw) 

 想写一个新的指南或贡献一个现有的？ 查看我们的[贡献指南](https://translate.googleusercontent.com/translate_c?depth=1&hl=zh-CN&rurl=translate.google.com&sl=auto&sp=nmt4&tl=zh-CN&u=https://github.com/spring-guides/getting-started-guides/wiki&usg=ALkJrhiB1b057IhnynHRSxHvX6ZckyPlzA) 。 

> 所有指南均附带代码的ASLv2许可证，以及用于撰写的[归属，NoDerivatives创意公共许可证](https://translate.googleusercontent.com/translate_c?depth=1&hl=zh-CN&rurl=translate.google.com&sl=auto&sp=nmt4&tl=zh-CN&u=https://creativecommons.org/licenses/by-nd/3.0/&usg=ALkJrhiBe5s-_7VG19vKBczGPd-MS34luA) 。

***

原文链接：[Building a RESTful Web Service](https://spring.io/guides/gs/rest-service/)