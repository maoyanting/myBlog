---
title: MySQL-基本操作
date: 2018-01-10 21:48:43
categories: 
- MySQL
tags: MySQL
---

本文整理自《深入浅出MySQL数据库开发、优化与管理维护（第2版）》

第一部分 基础篇

<!--more-->

## 数据库DCL

### **1、连接与断开MySQL服务器：**

```mysql
#启动mysql
mysql.server start
#回车后输入密码
mysql –uroot –p
#断开连接
quit或者exit
```

### **2、执行SQL语句：**

语句以“；”结束。不区分大小写。
在一行中可以输入多条SQL语句，各语句以“；”号隔开。
各条语句得到各自的结果集。
在任何时候输入“\c”并回车，则取消当前输入，重新回到mysql>提示符。
某些情况下，按回车后，SQL语句并不执行，出现的也不是“->”提示符，如：[’>]、[”>]、[`>]、[/*]，则表示引号或注释没有结束。即使通过\c取消，也要先结束引号。

### 3、创建数据库：

```mysql
#CHARACTER SET：字符集。
#COLLATE：整理或校对。如不指定则采用默认。
CREATE DATABASE db_test default character set gb2312 collate gb2312_chinese_ci;
```

### 4、关于字符校对collate

指比较字符串时采用的一套规则。还可以设置服务器校对和表校对。
一般情况下不需要设置字符校对，因为对应于每一种字符集MySQL有默认的校对规则，例如gb2312字符集默认校对规则为gb2312_chinese_ci。当不设置校对时采用默认。

### 5、删除**数据库：**

```mysql
DROP DATABASE database_name;
```

### 6、选择数据库：

```mysql
#在创建数据表之前，首先要选择数据库。
#其它查询、修改、删除等操作也一样。
use db_name;

#查看当前数据库下所有表：
show tables;
```

### 7、查看运行端口：

```mysql
show global variables like 'port';
```

### 8、执行sql脚本:

```mysql
source 文件位置
```

### 9、查看mysql版本

```mysql
#注意大写
mysql -V
```



------

## 表DDL

### 1、查看表：

```mysql
show columns from user_info from db_test;
show columns from db_test.user_info;

#如果当前使用数据库为表所在的数据库，则可以省略db_test。
describe user_info;

#用法同上，还可以只得到某一个列的信息：
describe user_info name;
```



### 2、删除表：

一次删除多个表：

```mysql
DROP TABLE user_info1，user_info2;
```

### 3、重命名表：

可以同时对多个表重命名，之间以逗号“，”隔开。

```mysql
RENAME TABLE user_info TO users_infomation, sale_info TO sales_infomation;
```

### 4、修改表：

| ALTER TABLE | table_name | MODIFY | [COLUMN] col_name | column_definition               | [FIRST/AFTER]col_name |
| ----------- | ---------- | ------ | ----------------- | ------------------------------- | --------------------- |
|             |            | ADD    | [COLUMN] col_name | column_definition               | [FIRST/AFTER]col_name |
|             |            | DROP   | [COLUMN] col_name |                                 |                       |
|             |            | CHANGE | [COLUMN] col_name | NEW col_name  column_definition | [FIRST/AFTER]col_name |
|             |            | RENAME | [TO]              | NEW table_name                  |                       |

modify和change区别：
change需要写两次col_name，但是可以用来修改列名。

在列reg_date中添加索引：

```mysql
ALTER TABLE user_information ADD INDEX (reg_date);
```

一次添加多个列(字段):

```mysql
ALTER TABLE table_name ADD func varchar(50), ADD gene varchar(50), ADD genedetail varchar(50);
```

###  

------

## 记录DML

### 1、插入记录：

```mysql
INSERT INTO table_name(…...) values (…..);

INSERT INTO user_info (name, gender, age, email) values(‘XiaoHei’, ‘男’，28，‘xhei@sohu.com’);

#可以一次同时插入多行记录，用“，”隔开。
INSERT INTO user_info(name,gender,age,email) values(‘ChenYi’, ‘女’，25，‘chenyi@shou.com’), (‘XiaoHei’, ‘男’，28，‘xhei@sohu.com’);
```

CONCAT为MySQL系统函数，将多个字符串连接起来。

### 2、修改记录：

```mysql
UPDATE table_name SET field1=values1,field2=values2,…. [ WHERE condition]

UPDATE table1_name t1, table2_name t2, table3_name t3  SET  t1.field1=values1,t2.field2=values2,…. [ WHERE condition]
```



### 3、删除记录：

```mysql
DELETE FROM table_name [ WHERE condition];

DELETE   t1,t2  FROM  table1_name t1, table2_name t2 [ WHERE condition];
```

各关键字含义用UPDATE，不指定where子句时删除所有行。

### 4、查询记录：

```mysql
#全选 SQL实际运行的时候会把*转换成列名，一般建议不使用*
SELECT * FROM  table_name  [ WHERE condition]; 

#选择部分字段
SELECT  field1_name,field2_name,…. FROM table_name  [ WHERE condition]; 

#去重 选择某字段的不重复的记录
SELECT  DISTINCT  field_name  FROM table_name  ； 

#排序 desc表示降序，默认为升序。
SELECT [ ORDER BY field1_name [DESC/ASC],  field2_name [DESC/ASC],  field3_name [DESC/ASC],….  ]    


#offset_start表示记录起始偏移量，
#row_count表示显示的行数。如果只写一个表示 row_count（其他数据库不通用）
#limit 和order by常配合使用来进行记录的分页显示。limit用在rollup后面。
SELECT [ LIMIT offset_start, row_count ]  

#GROUP BY 要进行分类聚合的字段
#WITH ROOLUP 是否对分类聚合后的结果进行再汇总，和ORDER BY是互斥的。
#HAVING 表示对分类后的结果再进行条件的过滤
#Function_name  ：sum，count（*），max，min
SELECT  field1_name,field2_name,…. Function_name FROM  table_name
[ WHERE condition][GROUP BY  field1_name,field2_name,…. [WITH ROOLUP] ][HAVING where_condition]
```



### 5、表连接p43

左连接：包含所有左边表中的记录甚至是右边表中没有和它匹配的记录
左连接：包含所有右边表中的记录甚至是左边表中没有和它匹配的记录

```mysql
SELECT col1_name,col2_name… FROM table1_name RIGHT/LEFT JOIN table2_name ON condition
```

### 6、子查询：

```mysql
WHERE col_name in (select …..)
```

如果子查询记录数唯一，可以用=代替in

### 7、记录联合：

UNION ALL 把结果直接合并在一起，
而UNION 是在此基础上进行一次DISTINCT去除重复记录后的结果。

------

## 踩坑的地方

1、sql语句中千万不要有sql的保留词，不然会无法识别
报错：`The error occurred while setting parameters`

2、MySQL和excel互相导入

excel到MySQL：excel文件另存为CSV utf-8（逗号分隔），MySQL直接import就行

MySQL到excel：MySQL 导出csv格式，excel选择导入。