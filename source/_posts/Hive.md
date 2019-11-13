---
title: Hive的使用
date: 2019.10.01 13:06
tags:
  - 大数据
description: Hive的使用
copyright: true
---

## Hive

- Hive由Facebook实现并开源
- 是基于Hadoop的一个数据仓库工具
- 可以将结构化的数据映射为一张数据库表
- 提供HQL(Hive SQL)查询功能
- 底层数据是存在HDFS上
- Hive的本质是将SQL语句转换为MapReduct任务进行
- 使得不熟悉MapReduce的用户很方便的利用HQL的处理和计算HDFS上的结构化的数据，适用于离线的批量数据计算。

#### 数据仓库（Data Warehouse）

是一个面向主题的，集成的Intergrated，相对稳定的Non-Volatile，反映历史变化的Time Variant的数据集合。

Hive依赖于HDFS存储数据，Hive将HQL转换成MapReduce执行，所以说Hive是基于Hadoop的一个数据仓库工具，实质上就是一个基于HDFS的MapReduce计算框架，对存储在HDFS中的数据进行分析和管理 。

#### 使用Hive

- 直接使用MapReduce的问题：
  - 人员学习成本太高
  - 项目周期要求太短
  - MapReduce实现复杂查询逻辑的开发难度太大

- 使用Hive
  - 更友好的接口：操作借口采用类SQL的语法，快速开发能力
  - 学习成本低：避免了写MapReduce，减少开发学习成本
  - 更好的扩展性：可自由扩展集群规模而不需要重启服务，用于还可以自定义函数

- Hive使用场景
  - Hive与传统的关系型SQL不同，支持绝大多数的语句DDL，DML以及聚集函数，连接查询，条件查询。
  - Hive不适合联机事务处理，不提供实时查询功能，适用于基于大量不可变的批处理作业。

- Hive的使用

  - 不支持Insert into，Update，Delete操作

  - 不支持等值连接

    ```sql
    select * from table a,table b where a.id=b.id;
    /*Hive中*/
    select * from table a join table b on a.id=b.id;
    ```

    