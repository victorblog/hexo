---
title: MySQL的问题
date: 2019.10.08 10:06
tags:
  - Java
description: MySQL的使用
copyright: true
---

### MySQL的问题

#### 1、重置id问题

由于在使用mysql，设计表的时候，设置了id自增，然后删除了数据后，再次新增数据时，就会出现id累计的情况，重置清空id，可以使用truncate

```mysql
#重置清空id，让id从1开始自增
truncate table t_student
```

#### 2、Insert ignore的使用

表要求有：primary key，或者有unique索引

Insert ignore会忽略已存在的数据

```mysql
insert ignore into t_student(name,age,class) values("test",19,"计算机");
```

