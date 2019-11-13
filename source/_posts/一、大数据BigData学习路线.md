---
title: 大数据
date: 2019.11.01 10:06
tags:
  - 大数据
description: 大数据
copyright: true
---

# 一、大数据BigData

### 1、简介

​	大数据本质也是数据，但是有了新的特征，包括数据来源广阔，数据格式多样化(结构化数据，非结构化数据，Excel文件，文本文件等)

数据量大(最少也是TB级别的，甚至可能是PB级别)，数据增长速度快。

### 2、需要考虑的问题

- 数据来源广，数据如何采集汇总？

  对应出现了Sqoop，Cammel，Datax等工具。

- 数据采集之后，如何存储？

  对应出现了GFS，HDFS，TFS等分布式文件存储系统

- 由于数据增长速度块，数据存储就必须水平扩展，于是出现集群

- 数据存储之后，该如何通过运算快速转化为统一的格式，该如何快速运算出自己想要的结果？

  - 对应的MapReduce，来解决这样的问题；但是写MapReduce需要写大量Java代码，所以出现了Hive，Pig等将SQL转化为MapReduce的解析引擎。

  - 但是普通的MapReduce处理数据只能一批一批的执行处理Job跑脚本，时间延迟太长，是离线计算，跑的都是前一天的数据。为了实现实时处理，每输入一条数据就能得到结果，于是出现了Storm/JStorm这样的低延时的流式计算框架。

  - 但是如果同时需要批处理和流处理，按照上面的，就需要搭建2个集群，一个Hadoop集群(包括HDFS+MapReduce+Yarn)和Storm集群。不便于管理，所以出现了Spark这样的一站式的计算框架，既可以进行批处理，也可以进行流处理(实质上是微批处理)。如今又进一步发展为Flink(快速流式处理计算框架)。

- 业务处理的通用架构是什么样的？
  - Lambda架构
  - Kappa机构出现

- 为了提高工作效率，加快运行速度，又出现了一些辅助工具
  - Ozzie，Azkaban：定时任务调度的工具
  - Hue，Zepplin：图形化任务执行管理，结果查看工具。类似公司的云窗系统
  - Scala语言：编写Spark程序的最佳语言，当然也可以使用Python或者Java
  - Allluxio，Kylin：通过对存储的数据进行预处理，就快了运算速度的工具，能够快速的查询得到结果。

### 3、大数据的必备技能

- 日志收集：Flume，Fluentd，ELK
- 流式计算：Storm/JStorm，Spark Streaming，Flink Streaming
- 编程语言：Java，Python，Scala
- 机器学习库：Mahout，MLib
- 消息队列：Kafka，RabbitMQ
- 数据分析/数据仓库：Hive，SparkSQL，FlinkSQL，Pig，Kylin
- Hadoop家族：Zookeeper，HBase，Hue，Sqoop，Oozie
- 资源调度：Yarn，Mesos。
- 分布式数据存储：HDFS
- 大数据通用处理平台：Hadoop，Spark，Flink

### 4、必须掌握的技能

- Java高级开发：JVM，并发
- Linux基本操作
- Hadoop（HDFS+MapReduce+Yarn）
- HBase(JavaAPI操作+Phoenix)
- Hive（Hql基本操作和原理理解）
- Kafka
- Storm
- Python
- Spark（Core+SparkSQL+Spark Streaming）
- 辅助小工具(Sqoop/Flume/jOzzie/Hue等)