---
title: "记一次jar包冲突导致的NoSuchMethodError"
date: 2019-04-10T08:40:46+08:00
draft: false
---

记一次jar包冲突导致的 `NoSuchMethodError`

<!--more-->

对接 `webService` 的时候，项目本身一直是用动态调用的，但是有部分代码出于技术原因使用的是生成的代码调用，然后引入了一个新的jar包

```xml
		<dependency>
			<groupId>axis</groupId>
			<artifactId>axis</artifactId>
			<version>1.4</version>
		</dependency>
```

运行新的代码没有问题，运行原有的动态调用的代码，就开始报错了。

```
java.lang.NoSuchMethodError: javax.wsdl.xml.WSDLReader.readWSDL(Ljavax/wsdl/xml/WSDLLocator;Lorg/w3c/dom/Element;)Ljavax/wsdl/Definition;
```

一般 `NoSuchMethodError` 通常就是 jar 包冲突了，我们搜索 `javax.wsdl.xml.WSDLReader` ，发现果然有两个



![全局搜索](https://pic-1256291953.cos.ap-shanghai.myqcloud.com/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20190410085511.png?q-sign-algorithm=sha1&q-ak=AKIDBNpxfmVdsFEuqgBMz7aLXKwKr8z8036j&q-sign-time=1554857716;1554859516&q-key-time=1554857716;1554859516&q-header-list=&q-url-param-list=&q-signature=d24a04dc0a701587f6903ae2947dce13d7d7c87c&x-cos-security-token=861736242294de818803fa7f25c80395b2ec0e3b10001)



我们继续查找一下我们需要调用的方法 `readWSDL` 在哪一个类里面。发现是新导入的包里面没有这个方法，然后我们把新导入的包里面这个jar包剔除掉

```xml
		<dependency>
			<groupId>axis</groupId>
			<artifactId>axis</artifactId>
			<version>1.4</version>
			<exclusions>
				<exclusion>
					<artifactId>axis-wsdl4j</artifactId>
					<groupId>axis</groupId>
				</exclusion>
			</exclusions>
		</dependency>
```

再启动一下，两个都能正常运行啦