---
title: HBase
date: 2019-11-25 11:24:39
tags:
- 大数据
toc: true
copyright: true
---

# 一、背景

​		由于关系型数据库用来解决数据存储和维护有关的问题。大数据出现后，大多公司实现处理大数据，并开始选择Hadoop的解决方案。Hadoop使用分布式文件系统HDFS，用来存储大数据，使用MapReduce来处理数据。

​		Hadoop只能执行批量处理，而且是按顺序访问数据。也就是表明必须搜索整个数据集。如果想要处理结果在另一个庞大的数据，也是按照顺序处理一个巨大的数据集。为此，需要访问数据中的任何点的单元，也就是随机访问，出现新的解决方案。

​		Hadoop随机存取数据库的应用，HBase，Cassandra，CouchDB，Dynamo，MongoDB都是一些存储大量数据和随机访问访问的非关系型数据库。

- 海量数据存储成为瓶颈，单机无法负载大量数据
- 单机IO读写请求成为海量数据存储，高并发，大规模的瓶颈
- 随着数据规模越来越大，大量业务场景开始考虑数据存储横向水平扩展，让存储服务可以增加/删除 ，而大部分关系型数据库更专注在一台机器。

# 二、HBase简介

​		HBase是BigTable的开源版本，使用Java编写的。是Apache Hadoop的数据库，建立在HDFS上，用来提供，高可靠，高性能，列存储，可伸缩，多版本的NoSQL的分布式数据库。可以实现对大型数据的实时、随机的读写访问。

- HBase依赖HDFS做底层的数据存储，BigTable依赖Google GFS做数据存储
- HBase依赖MapReduce进行数据计算，BigTable依赖Google MapReduce做数据计算
- HBase依赖Zookeeper做服务协调，BigTable依赖Google Chubby做服务协调

### 1、非关系型和关系型数据库

- NoSQL：HBase，Redis，MongoDB
- RDBMS：MySQL，Oracle，SQL Server

### 2、HBase重点

- HBase介于NoSQL和RDBMS之间，只能通过主键rowkey和主键的range来检索数据
- HBase查询数据功能很简单，不支持join等关键查询复杂操作，可以通过Hive支持来实现多表join
- HBase不支持复杂的事务，只支持行级事务
- HBase中纯支持的数据类型，byte[]，底层所有数据的存储都是字节数组
- HBase主要来存储结构化和半结构化的松散数据
  - 结构化数据：数据结构字段清晰，比如数据库的表结构
  - 半结构化：具有一定结构，语义不太确定，比如html网页的数据。

### 3、HBase的表

- 数据量大：一张表可以有上十亿行，上百万列的数据
- 面向列：列（族）的存储和权限控制，列簇独立检索
- 稀疏数据：对于null的空列，并不会占用存储空间，在设计表的时候可以非常稀疏
- 无模式：每行都有一个可以排序的主键rowkey和任意多的列，列可以根据需要动态的增加，同一张表中不同的行可以有截然不同的列

| Rowkey | ColumnFamily | :CF1         | ColumnFamily | :CF2       | TimeStamp |
| :----: | -----------: | :----------- | -----------: | :--------- | :-------: |
|        |  Column:Name | Column:Alias |   Column:Age | Column:Sex |           |
| rk001  |     zhangsan | shanghai     |           22 | M          |    T1     |
| rk002  |         lisi | beijing      |           25 | F          |    T2     |

#### 3.1、RowKey的含义

##### 3.1.1、rowkey

​		HBase中的rowkey和MySQL中的主键是完全一样的，来唯一的区分某一行的数据。

##### 3.1.2、HBase支持3种查询方式：

- 基于Rowkey单行查询
- 基于Rowkey的范围扫描
- 全表扫描

##### 3.1.3、rowkey分析

- rowkey对HBase的性能影响很大，rowkey的设计也很重要，设计的时候需要考虑基于rowkey单行查询，还要考虑rowkey的范围查询。

- rowkey行主键，可以是任意字符串，最大长度为64kb，实际中一般是10~100b，最好是16。HBase内部，rowkey保存为字节数组。HBase会对表中的数据按照rowkey进行字典序排序

#### 3.2、Column的含义

- 列，可以理解为MySQL的列

#### 3.3、ColumnFamily的含义

- 列族，HBase引入的定义
- HBase通过列族划分数据的存储，列族下面可以包含若干个列，实现灵活的数据存取。就如同一个家族一样，列族是由一个个列组成。
- HBase的表在创建的时候必须指定列族，就像你在MySQL创建表的时候必须指定具体的列一样。
- HBase的列族并不是越多越好，一般列族最好是小于等于3个，一般常用1个列族

#### 3.4、TimeStamp的含义

- TimeStamp是实现HBase多版本的关键。HBase中使用不同的TimeStamp来标识相同rowkey行对应的不同版本的数据。类似Hive表中的时间分区字段。
- HBase中通过rowkey和column确定一个存储单元为cell，每个cell保存同一份数据的多个版本。版本通过时间戳来索引。时间戳可以在HBase写入数据的时候自动赋值。为了避免数据版本冲突，必须自己生成具有唯一性的时间戳。一般每个cell中，不同版本的数据按照时间倒序，最新的数据在最前面。
- 为了避免数据存太多版本造成存储和索引的负担，HBase还提供了2种版本回收方式
  - 保存数据的最后n个版本
  - 保存最近一段时间内的版本(可以设置数据的生命周期TTL)

#### 3.5、单元格Cell

- 由rowkey，column，version唯一确定的单元。Cell中的数据是没有类型的，全部是字节码byte[]形式存储。