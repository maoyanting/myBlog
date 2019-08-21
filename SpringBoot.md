---
title: "Spring Boot(未完)"
date: 2019-02-25T22:12:50+08:00
---



这年头新项目不用 `Spring Boot` 的大概想法比较异于常人😅

代码基于 `Spring Boot 2.0`

<!--more-->



### SpringApplication启动

一般一个module Spring Boot 都会生成一个启动类，类似于下面

```java
@SpringBootApplication
@MapperScan("com.yuantu.billmanager.mapper")
public class WebApplication {

    public static void main(String[] args) {
        SpringApplication.run(WebApplication.class, args);
    }
}
```



我们首先进入 `SpringApplication.run` 这里调用了另一个  `SpringApplication.run`

```java
public static ConfigurableApplicationContext run(Class<?> primarySource,String... args) {
    return run(new Class<?>[] { primarySource }, args);
}
```

继续下一个方法，这个 `run` 方法里面是可以放入多个 `primarySource` 的，具体使用情况我们以后再看

这里调用了 `SpringApplication`  的构造函数，然后又运行 `run` 方法（妈耶你怎么还没执行😰）

```java
public static ConfigurableApplicationContext run(Class<?>[] primarySources,
			String[] args) {
    return new SpringApplication(primarySources).run(args);
}
```

我们先来看构造函数，再往下，我们看看之前 `Spring` 让人崩溃且记不住的配置到底怎么处理了。 

```java
public SpringApplication(Class<?>... primarySources) {
    this(null, primarySources);
}

@SuppressWarnings({ "unchecked", "rawtypes" })
public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
    this.resourceLoader = resourceLoader;
    //校验
    Assert.notNull(primarySources, "PrimarySources must not be null");
    //保存primarySources
    this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));
    //判断是否是web应用
    this.webApplicationType = deduceWebApplicationType();
    setInitializers((Collection) getSpringFactoriesInstances(
				ApplicationContextInitializer.class));
    //寻找Listener
    setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
    //找到含有main函数的类
    this.mainApplicationClass = deduceMainApplicationClass();
}
```

构造完毕以后，我们终于要执行 `run` 方法了

```java
public ConfigurableApplicationContext run(String... args) {
    //Spring提供的计时器
    StopWatch stopWatch = new StopWatch();
    stopWatch.start();
    ConfigurableApplicationContext context = null;
    Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList<>();
    //配置Headless，告诉程序我没有显示器等硬件
    configureHeadlessProperty();
    //获取Listeners
    SpringApplicationRunListeners listeners = getRunListeners(args);
    listeners.starting();
    try {
        ApplicationArguments applicationArguments = new DefaultApplicationArguments(
            args);
        //准备环境
        ConfigurableEnvironment environment = prepareEnvironment(listeners,
            applicationArguments);
        //配置ignore，默认为true，不扫描BeanInfo类
        configureIgnoreBeanInfo(environment);
        //绘制Banner，这里可以自己生成一个Banner.txt文件来代替
        Banner printedBanner = printBanner(environment);
        //生成上下文容器
        context = createApplicationContext();
        exceptionReporters = getSpringFactoriesInstances(
            SpringBootExceptionReporter.class,
            new Class[] { ConfigurableApplicationContext.class }, context);
        prepareContext(context, environment, listeners, applicationArguments,
	            printedBanner);
        //刷新容器
        refreshContext(context);
        //里面是空的……空的……大概可以用来实现一些特殊需求
        afterRefresh(context, applicationArguments);
        //停止计时器
        stopWatch.stop();
        if (this.logStartupInfo) {
            new StartupInfoLogger(this.mainApplicationClass)
                .logStarted(getApplicationLog(), stopWatch);
        }
        listeners.started(context);
        //这里查找实现了ApplicationRunner和CommandLineRunner的类，可以再这两个类里面加希望Spring初始化以后执行的内容，SpringMVC的话一般使用Listener来实现。
        callRunners(context, applicationArguments);
        }catch (Throwable ex) {
            handleRunFailure(context, listeners, exceptionReporters, ex);
            throw new IllegalStateException(ex);
        }
        listeners.running(context);
        return context;
    }
```

