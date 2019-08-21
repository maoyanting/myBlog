---
title: Spring-全局异常拦截
date: 2018-10-16 15:49:06
categories: 
- Spring
tags:
- Spring
---



Spring MVC那一篇里提到了异常拦截来做参数校验返回，那里是对特定的 `controller` 做异常捕捉，但是我们也可以选择全局拦截处理

<!--more-->



## 快速开始



```java
@ResponseBody
@ControllerAdvice
public class ExceptionAdvice {
    private static Logger logger = LoggerFactory.getLogger(ExceptionAdvice.class);

    /***
    * 参数绑定异常
    * @date 2018/10/16
    * @param exception HttpMessageNotReadableException
    */
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    @ExceptionHandler(HttpMessageNotReadableException.class)
    public Result<Long> messageNotReadable(HttpMessageNotReadableException exception){
        InvalidFormatException formatException = (InvalidFormatException)exception.getCause();
        List<JsonMappingException.Reference> e = formatException.getPath();
        String fieldName = "";
        for (JsonMappingException.Reference reference :e){
            String fieldName = reference.getFieldName();
        }
        logger.error("参数不匹配"+exception);
        return Result.createFailResult(fieldName+"参数类型不匹配");
    }

    /***
    * 全局异常，如果没有匹配到上述准确的异常，都会到这里来处理
    * @date 2018/10/16
    * @param e 没有匹配到的全局异常
    */
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    @ExceptionHandler(Exception.class)
    public Result<String> all(Exception e){
        //这里的log使用了“，”，这样能把异常的堆栈信息全部打印出来，更容易定位bug
        logger.error("异常：",e);
        return Result.createFailResult("工程抢救中……请稍后再试");
    }
}
```



## @ControllerAdvice

`@ControllerAdvice` 默认监控所有的 `@RequestMapping` 方法，也可以对指定过滤的条件：

```java
// 监控所有的被@RestController注解的Controllers类 
@ControllerAdvice(annotations = RestController.class)

// 监控特定的包下的Controllers类
@ControllerAdvice("org.example.controllers")

// 监控指定类的Controllers类
@ControllerAdvice(assignableTypes = {ControllerInterface.class, AbstractController.class})

```





