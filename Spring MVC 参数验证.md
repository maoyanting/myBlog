---
title: Spring MVC 参数验证
date: 2018-09-10 23:46:11
tags:
---



参数校验也可以很简单啊

<!--more-->



## 零、使用

> 参考资料：[【SpringMVC学习06】SpringMVC中的数据校验 ](https://blog.csdn.net/eson_15/article/details/51725470)

spring boot默认使用 hibernate validator



一、依赖（以spring boot 为例）

```xml
        <dependency>
            <groupId>javax.validation</groupId>
            <artifactId>validation-api</artifactId>
            <version>RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.hibernate.validator</groupId>
            <artifactId>hibernate-validator</artifactId>
            <version>RELEASE</version>
        </dependency>
```

二、在你需要校验的属性上加上@NotNull等注释

```java
public class Person{
    @NotNull
	private String name;  
}
```

三、加上@Valid注释，同时用bindingResult获取异常（必须紧跟在@Valid修饰的变量后面，这是因为，在参数列表中允许有多于一个的模型对象，Spring会为它们创建不同的`BindingResult`实例。 ）

```java
	@RequestMapping(value = "/edit")
    public Result<Long> insertBase(@Valid Person person, BindingResult bindingResult) {
        //参数校验
        if (bindingResult.hasErrors()) {
            return Result.createFailResult(ResultCodeEnums.PARAM_ERROR);
        }
	}
```



## 一、规则



| @AssertFalse                                 | Boolean,boolean                                              | 验证注解的元素值是false                                      |
| -------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| @AssertTrue                                  | Boolean,boolean                                              | 验证注解的元素值是true                                       |
| @NotNull                                     | 任意类型                                                     | 验证注解的元素值不是null                                     |
| @Null                                        | 任意类型                                                     | 验证注解的元素值是null                                       |
| @Min(value=值)                               | BigDecimal，BigInteger, byte,short, int, long，等任何Number或CharSequence（存储的是数字）子类型 | 验证注解的元素值大于等于@Min指定的value值                    |
| @Max（value=值）                             | 和@Min要求一样                                               | 验证注解的元素值小于等于@Max指定的value值                    |
| @DecimalMin(value=值)                        | 和@Min要求一样                                               | 验证注解的元素值大于等于@ DecimalMin指定的value值            |
| @DecimalMax(value=值)                        | 和@Min要求一样                                               | 验证注解的元素值小于等于@ DecimalMax指定的value值            |
| @Digits(integer=整数位数, fraction=小数位数) | 和@Min要求一样                                               | 验证注解的元素值的整数位数和小数位数上限                     |
| @Size(min=下限, max=上限)                    | 字符串、Collection、Map、数组等                              | 验证注解的元素值的在min和max（包含）指定区间之内，如字符长度、集合大小 |
| @Past                                        | java.util.Date,java.util.Calendar;Joda Time类库的日期类型    | 验证注解的元素值（日期类型）比当前时间早                     |
| @Future                                      | 与@Past要求一样                                              | 验证注解的元素值（日期类型）比当前时间晚                     |
| @NotBlank                                    | CharSequence子类型                                           | 验证注解的元素值不为空（不为null、去除首位空格后长度为0），不同于@NotEmpty，@NotBlank只应用于字符串且在比较时会去除字符串的首位空格 |
| @Length(min=下限, max=上限)                  | CharSequence子类型                                           | 验证注解的元素值长度在min和max区间内                         |
| @NotEmpty                                    | CharSequence子类型、Collection、Map、数组                    | 验证注解的元素值不为null且不为空（字符串长度不为0、集合大小不为0） |
| @Range(min=最小值, max=最大值)               | BigDecimal,BigInteger,CharSequence, byte, short, int, long等原子类型和包装类型 | 验证注解的元素值在最小值和最大值之间                         |
| @Email(regexp=正则表达式,flag=标志的模式)    | CharSequence子类型（如String）                               | 验证注解的元素值是Email，也可以通过regexp和flag指定自定义的email格式 |
| @Pattern(regexp=正则表达式,flag=标志的模式)  | String，任何CharSequence的子类型                             | 验证注解的元素值与指定的正则表达式匹配                       |
| @Valid                                       | 任何非原子类型                                               | 指定递归验证关联的对象；如用户对象中有个地址对象属性，如果想在验证用户对象时一起验证地址对象的话，在地址对象上加@Valid注解即可级联验证 |



> 参考资料：[SpringMVC数据验证——第七章 注解式控制器的数据验证、类型转换及格式化——跟着开涛学SpringMVC](http://jinnianshilongnian.iteye.com/blog/1733708)



## 二、Validator

> 参考资料：[使用Validator做SpringMVC的验证框架 - 使用Validator](https://blog.csdn.net/devefx/article/details/51567533)



## 三、**@Validated** 和@Valid的区别

原文链接：https://stackoverflow.com/a/52092984/8147625

 **@Valid的使用**

Controller:

```java
@RequestMapping(value = "createAccount")
public String stepOne(@Valid Account account) {...}
```

Form object:

```java
public class Account {

    @NotBlank
    private String username;

    @Email
    @NotBlank
    private String email;

}
```

------

**@Validated的使用**
Source: <http://blog.codeleak.pl/2014/08/validation-groups-in-spring-mvc.html>

Controller:

```java
@RequestMapping(value = "stepOne")
public String stepOne(@Validated(Account.ValidationStepOne.class) Account account) {...}

@RequestMapping(value = "stepTwo")
public String stepTwo(@Validated(Account.ValidationStepTwo.class) Account account) {...}
```

Form object:

```java
public class Account {

    @NotBlank(groups = {ValidationStepOne.class})
    private String username;

    @Email(groups = {ValidationStepOne.class})
    @NotBlank(groups = {ValidationStepOne.class})
    private String email;

    @NotBlank(groups = {ValidationStepTwo.class})
    @StrongPassword(groups = {ValidationStepTwo.class})
    private String password;

    @NotBlank(groups = {ValidationStepTwo.class})
    private String confirmedPassword;
    
    public interface ValidationStepOne {
    }
    public interface ValidationStepTwo {
    }

}
```





## 四、常见问题

### 1、@Valid在list失效

你看一下你是不是没有在list之前加上` @Valid`

### 2、参数绑定的error

如果添加的注解在参数绑定的时候就报错了，比如需要的是Integer，但是传入的是String

这时候分三种情况

1、有`@Valid` 有`BindingResult` 使用的是普通的参数接收；

可以被`bindingResult`捕获，判断`isBindingFailure`可以知道是否是参数绑定失败

此时你添加在这个参数上的校验注解会不启用，因为参数绑定是在注解校验之前

```java

	@RequestMapping(value = "/edit")
    public Result<Long> insertBase(@Valid Person person, BindingResult bindingResult) {
        Result<Long> result = Result.createFailResult();
        //参数校验
        if (bindingResult.hasErrors()) {
            List<FieldError> list = bindingResult.getFieldErrors();
            String message = list.get(0).getDefaultMessage();
            if (list.get(0).isBindingFailure()){
                message = HealthRecordVO.getValidName().get(list.get(0).getField())+"类型不匹配";
            }
            result.setMsg(message);
            return result;
        }
	}
	
	
// 此时getDefaultMessage会显示 ：Failed to convert property value of type 'java.lang.String' to required type 'java.lang.Integer' for property 'weight'; nested exception is java.lang.NumberFormatException: For input string: "字符串"
```

2、有`@Valid` 有`BindingResult` 参数是以json形式接收的；

此时就不能被`bindingResult`捕获了，会直接抛给前端400的异常

```java
@RequestMapping(value = "/edit", method = RequestMethod.POST)
public Result<Long> insertBase(@RequestBody @Valid Person person, BindingResult bindingResult) {
    Result<Long> result = Result.createFailResult();
    //参数校验
    if (bindingResult.hasErrors()) {
        List<FieldError> list = bindingResult.getFieldErrors();
        String message = list.get(0).getDefaultMessage();
        if (list.get(0).isBindingFailure()){
            message = HealthRecordVO.getValidName().get(list.get(0).getField())+"类型不匹配";
        }
        result.setMsg(message);
        return result;
    }
}
//此时bindingResult啥也拿不到
```
一般来说类型匹配应该是由前端来做控制，但是保不齐有什么特殊情况（比如前端实在是没有时间……）

此时我们可以在这个controller中加上一个单独的异常处理

```java
    @ResponseBody
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    @ExceptionHandler(HttpMessageNotReadableException.class)
    public Result<Long> messageNotReadable(HttpMessageNotReadableException exception){
        InvalidFormatException formatException = (InvalidFormatException)exception.getCause();
        List<JsonMappingException.Reference> e = formatException.getPath();
        String fieldName = "";
        for (JsonMappingException.Reference reference :e){
            //这里获得了类型匹配出错的属性的属性名
            String field = reference.getFieldName();
        }
        return Result.createFailResult(field+"请输入数字");
    }
```

` @ResponseStatus` 这个注解是为了标识这个方法返回值的HttpStatus code ，这里我们不变，还是抛出400

 `@ExceptionHandler ` 当一个Controller中有方法加了@ExceptionHandler之后，这个Controller其他方法中没有捕获的异常就会以参数的形式传入加了@ExceptionHandler注解的那个方法中。 

在这里，由于springmvc默认采用jackson作为json序列化工具，当反序列化失败的时候就会抛出HttpMessageNotReadableException

当然这里只针对了一个controller里面的参数校验，你同样可以使用`@ControllerAdvice `来做全局异常判断

3、没有`@Valid` 或者没有`BindingResult`

此时接口返回的就是400了，毕竟你啥也没做

> 参考链接：https://www.cnblogs.com/woshimrf/p/spring-web-400.html