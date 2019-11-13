---
title: UDF自定义函数
date: 2019.11.01 10:06
tags:
  - 大数据
description: UDF自定义函数
copyright: true
---

# 一、UDF的使用手册

### 1、简介

Hive中，提供了丰富的内置函数，比如<font color=red>trim(),cast(),max(),count(),coalesce()</font>等之外，    还允许用户用java开发自定义的UDF函数。

#### 1.1、开发自定义UDF函数的2种方式：

- 继承org.apache.hadoop.hive.ql.exec.UDF;
- 继承org.apache.hadoop.hive.ql.udf.generic.GenericUDF; 

#### 1.2、总结：

- 针对简单数据类型：<font color=red>String,Integer等，可以使用UDF</font>。
- 针对复杂数据类型：<font color=red>Array,Map,Struct等，可以使用GenericUDF</font>
- GenericUDF还可以在函数开始之前和结束之后做一些初始化和关闭的处理操作。

### 2、UDF

使用UDF实现对String类型的字符串取HashMD5

```java

```



### 3、GenericUDF

