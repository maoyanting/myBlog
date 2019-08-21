---
title: "LittleTips-一些小技巧集合"
date: 2019-03-25T22:07:05+08:00
---





一些很小的技巧合集

<!--more-->

### .properties文件中定义数组

如果你在 .properties文件中定义数组： 

```
base.module.elementToSearch=1,2,3,4,5,6
```

你可以在你的Java类中加载这样的数组，像这样：

```
  @Value("${base.module.elementToSearch}")
  private String[] elementToSearch;
```

> 参考链接：[Spring .properties file: get element as an Array](https://stackoverflow.com/questions/6212898/spring-properties-file-get-element-as-an-array)

ps：实测在Disconf没有用