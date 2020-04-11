---
title: Spark Streaming的使用
date: 2018.10.05 10:06
tags:
  - 大数据
description: Spark Streaming的使用
copyright: true
toc: true
---

## 一、Spark Streaming

### 1、使用背景

​		有时候需要实时处理收到的数据，比如实时追踪页面访问统计的应用，训练机器学习模型，自动检测异常等。Spark Streaming是Spark为这些应用而设计的模型。让用户使用一套和批处理非常接近的API来编写流式计算应用，可以大量重用批处理应用的技术和代码。

​		和Spark基于RDD的类似，Spark Streaming使用离散化流作为抽象表示，叫做DStream。而DStream是随时间推移而收到的数据的序列。DStream内部每个时间区间收到的数据都作为RDD的存在，DStream是由这些RDD所组成的序列，因此是离散化。DStream可以从各种输入源创建，常见的是Flume，Kafka，HDFS。创建出来的DStream支持2种操作，一种是转化操作Transformation，会生成新的DStream。一种是输出操作Action。可以把数据写入外部系统中。DStream提供了许多与RDD所支持的操作，同时还增加了与时间相关的新操作，滑动窗口。

​		与批处理Job不同的是，Spark Streaming应用需要进行额外配置来保证24小时不间断工作。比如checkpoint机制，也就是把数据存储到可靠文件系统HDFS上的机制，这也是Spark Streaming用来实现不间断工作的主要方式。

### 2、实例

​		Spark Streaming程序最好是使用Maven 或者Sbt编译的独立应用形式运行。Spark Streaming虽然是Spark的一部分。

- pom.xml依赖文件

  ```xml
  <dependency>
              <groupId>org.apache.spark</groupId>
              <artifactId>spark-streaming_2.10/artifactId>
              <version>1.2.0</version>
          </dependency>
  ```

- Java流计算import

