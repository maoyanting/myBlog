---
title: Spring-sleuth
date: 2018-10-22 22:49:08
categories: 
- Spring
tags:
- Spring
- Spring Cloud
---

有时候，定位bug真的是个很麻烦的事情呢

我们可以简单借用一下Spring Cloud Sleuth

<!--more-->



## 快速开始



```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-sleuth</artifactId>
            <version>2.0.1.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-sleuth</artifactId>
    </dependency>
</dependencies>
```

这里注意一下 `version`  ，有时候会导致依赖进不来

好了，这就可以了😜，真的非常简单而且没有侵入性

此时打印出来的日志大概是这样的

```
2018-10-22 23:03:14.029  INFO [-,,,] 12936 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8081 (http) with context path '/sandao'
2018-10-22 23:03:14.039  INFO [-,,,] 12936 --- [           main] com.sandao.DemoSandaoApplication         : Started DemoSandaoApplication in 9.364 seconds (JVM running for 10.336)
```

这里来因为是启动日志，所以没有traceId和spanId

这里我们借用启动类调用一下接口

```java
@RestController
@SpringBootApplication(exclude = DataSourceAutoConfiguration.class)
public class DemoSandaoApplication {

	public static void main(String[] args) {
		SpringApplication.run(DemoSandaoApplication.class, args);
	}

	private static final Logger logger = LoggerFactory.getLogger(DemoSandaoApplication.class);

	@RequestMapping("/log")
	public String home() {
		logger.info("Handling home");
		return "Hello World";
	}
}
```

打印出来的日志大概是这样的

```
2018-10-22 23:06:29.838  INFO [-,30f87869bf24ede4,30f87869bf24ede4,false] 12936 --- [nio-8081-exec-2] com.sandao.DemoSandaoApplication         : Handling home
```

我们就可以看见两个id了



***

有些教程会建议你（包括官方文档）在 `application.yml` / `application.properties` 里面加上项目的名称

比如 `application.yml`：

```yaml
spring:
  application:
    name: sandao
```



这时候打印出来的日志大概是这样的

```
2018-10-22 23:00:23.209  INFO [sandao,,,] 9280 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8081 (http) with context path '/sandao'
2018-10-22 23:00:23.221  INFO [sandao,,,] 9280 --- [           main] com.sandao.DemoSandaoApplication         : Started DemoSandaoApplication in 17.512 seconds (JVM running for 19.017)
```



## 为什么日志没有打印出来id

首先看一下你的依赖导进来没有

然后你看一下你是不是配置了日志的格式，如果配置了日志的格式，就需要显式地写出来 `traceId` 和 `spanId` ，我当时被这个坑了很久……

```xml
<configuration>
    <!-- TraceId:一个请求一个id -->
    <property name="CONSOLE_LOG_PATTERN"
              value="%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level [ %X{X-B3-TraceId:-} %X{X-B3-SpanId:-}] %logger{5} - %msg%n"/>

    <appender name="rollingAppender" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${LOG_PATH}/doct.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>/logs/heuristic-%d{yyyy-MM-dd}.log</fileNamePattern>
            <maxHistory>30</maxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>${CONSOLE_LOG_PATTERN}</pattern>
            <charset>utf8</charset>
        </encoder>
        <append>false</append>
        <prudent>false</prudent>
    </appender>

    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>${CONSOLE_LOG_PATTERN}</pattern>
            <charset>utf8</charset>
        </encoder>
    </appender>

    <root level="info">
        <appender-ref ref="STDOUT"/>
        <appender-ref ref="rollingAppender"/>
    </root>

</configuration>
```



## 获取当前的traceId

`sleuth` 多用于分部署系统，和 `zipkin` 配合使用，不过有时候我们可能也需要在项目中获取traceId，可以参考下述的方式。

```java
import brave.Tracer;

@Service
public class Breadcrumb {

  @Autowired
  private Tracer tracer;

  public String breadcrumbId() {
    return tracer.currentSpan().context().traceIdString();
  }
}
```