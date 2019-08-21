---
title: Mybatis-小问题
date: 2018-09-10 23:03:31
tags: Mybatis
---



## 一、Parameter 'id' not found. Available parameters are [arg2, arg1, arg0, param3, param1, param2]

原因：多入参的情况下没有在参数前加@Param

特殊原因：加了@Param还是报错，查看你是import的依赖是不是`org.apache.ibatis.annotations.Param`

可能会不小心导入`org.springframework.data.repository.query.Param`