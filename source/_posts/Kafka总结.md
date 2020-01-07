---
title: 分布式锁
date: 2018.07.15 10:06
tags:
  - 大数据
description: Kafka的总结
toc: true
copyright: true
---

# 1、Kafka有哪些特点？

- 高吞吐量、低延迟：Kafka每秒可以处理几十万条消息，它的延迟最低只有几ms，每个topic可以分多个partition，Consumer group对partition进行consume操作。
- 可扩展性：Kafka集群支持热扩展
- 持久性、可靠性：消息被持久化到本地磁盘，并且支持数据备份防止数据丢失。
- 容错性：允许集群中节点失败，加入副本数为n，则允许n-1个节点失败
- 高并发：支持数千个客户端同时读写

# 2、哪些场景下会选择Kafka？

- 日志收集：公司用Kafka手机各种服务的log，通过Kafka以统一接口服务的方式开放给各种consumer，比如hadoop，HBase，Spark，Flink等
- 消息系统：解耦生产者和消费者，缓存消息
- 用户活动跟踪：Kafka经常被用来记录web用户或者app用户的各种活动，比如浏览网页，搜索，点击等活动，这些活动信息被各个服务器发布到kafka的topic中，然后订阅者通过订阅这些topic来做实时的监控分析，或者落到HDFS，数据仓库中做离线分析和挖掘。
- 运营指标：Kafka经常用来记录运营监控数据。包括收集各种分布式应用的数据，生产各种操作的集中反馈，比如报警和报告。
- 流式处理：spark streaming和Flink

# 3、Kafka分区的目的？

partition分区对于kafka集群的好处是：实现负载均衡。分区对于消费者来说，可以提高并发度，提高效率。

# 4、Kafka是如何做到消息的有序性？

kafka中的每个partition中的消息在写入的时候都是有序的，而且单独一个partition只能由一个消费者去消费。可以在里面保证消息的顺序性，但是分区之间的消息不保证有序的。

# 5、Kafka是如何保证数据可靠性和一致性？

### 5.1、数据可靠性

#### 1、Topic分区副本

​		在kafka0.8.0之前，kafka是没有副本partition的概念的，那时人们只会用Kafka存储一些不重要的数据，因为没有副本，数据可能会丢失。但是随着业务发展，支持副本的功能越来越强大，为了保证数据的可靠性，kafka从0.8.0版本开始引入了分区副本。也就是每个分区可以人为的配置几个副本，比如在创建主题的时候指定replication-factor，也可以在Broker级别进行配置default.replication.factor，一般会设置为3个副本。

​		kafka可以保证单个分区里的消息是有序的，分袂可以在线可用，可以离线不可用。在多个分区副本里面有一个副本是leader，其余的副本是follower，所有的读写操作都是经过leader进行的，同时follower会定期的去leader上复制数据。当leader挂了，其中一个follower会重新成为新的leader，通过分区副本，引入了数据冗余，同时提供了kafka的数据可靠性。

​		kafka的分区副本架构是kafka可靠性保证的核心，把消息写入多个副本可以让kafka在发生崩溃的时候仍能保证消息的持久性。

#### 2、Producer往Broker发送消息

​		如果我们要往kafka对应的topic发送消息，我们需要通过Producer完成，kafka主题对应了多个分区partition，每个分区下面又对应了多个副本。为了让用户设置数据可靠性，Kafka在Producer里面提供了消息确认机制。也就是说我们可以通过配置来决定消息发送到对应分区的几个副本才算消息发送成功。可以在定义Producer时通过acks参数指定。这个acks参数支持3个值：

- acks=0：表示如果生产者能够通过网络把消息发送出去，那么就任务消息已经成功写入kafka。这种情况下还是有可能发生错误，比如发送的对象不能被序列化或者网卡发生故障，但如果是分区离线或者整个集群长时间不可用，那么就不会收到任何错误。在acks=0模式下的运行速度是非常快的，很多基准测试都是这个模式，你可以得到惊人的吞吐量和带宽利用率，不过这种模式，一定会丢失一些消息。
- acks=1:意味着如果leader在收到消息并把消息写到分区数据文件，并不一定同步到磁盘上时，会返回确认或者错误响应。这个模式下，如果发生正常的leader选举，生产者会在选举时收到一个LeaderNotAvaiableException异常，如果生产者能恰当的处理这个错误，它会重试发送消息，最终消息会安全到达新的Leader里。不过在这个模式下仍然可能丢失数据，比如消息已经成功写入Leader，但是在消息被赋值到follower副本之前Leader发生崩溃。
- acks=all:这个和旧版本参数request.required.acks=-1的意义一样：意味着Leader在返回确认或者错误响应之前，会等待所有同步副本都受到消息，如果和min.insync.replicas参数结合起来。就可以决定在返回确认前至少有多少个副本能够收到消息，生产者会一直重试直到消息被成功提交。不过这也是最慢的做法，因为生产者在继续发送其他消息之前需要等待所有副本都收到当前的消息。

Producer发送消息还可以选择同步，producer.type=sync默认是同步。也可以配置成异步producer..type=async模式。如果设置成异步，虽然会极大的提高消息发送性能，但是这样会增加丢数据的风险。如果需要确保消息的可靠性，必须将producer.type=sync。

#### 3、Leader选举

​		ISR（in-sync replicas）列表。每个分区的leader会维护一个ISR列表，ISR列表里面就是follower副本的Broker编号，之后跟的上Leader的follower副本才能加入到ISR里面，这个是通过replica.lag.time.max.ms参数配置的，只有ISR里的成员才有被选举为Leader的可能。

​		当Leader 挂了，而且unclean.leader.election.enable=false情况下，kafka会从ISR列表中选择第一个follower作为新的leader，因为这个分区拥有最新的已经commit的消息，通过这个可以保证赢commit的消息的数据可靠性。

​		为了保证数据的可靠性，至少需要配置以下几个参数：

- producer级别：acks=all(旧版本request.required.acks=-1)，同时发生模式改为同步producer.type=sync
- topic级别：设置replication.factor>=3,并且min.insync.replicas>=2
- broker级别：关闭不完全的leader选举，unclean.leader.election.enable=false;

### 5.2、数据一致性

​	此处的数据一致性是不论新的Leader还是新选举的Leader，consumer都能督导一样的数据。

​		假如分区的副本partition为3，其中副本0是leader，副本1和副本2是follower，并且在ISR列表里面，虽然副本0已经写入了Message4，但是Consumer只能读到Message2。因为所有的ISR都同步了Message2，只有High Water Mark以上的消息才支持Consumer读取，而High Water Mark取决于ISR列表里面的偏移量最小的分区。类似木桶效应。

​		这样做的原因是还没有被足够多副本赋值的消息被认为是不安全的，如果Leader发生崩溃，另一个副本成为新Leader，那么这些消息很可能丢失了。如果我们允许消费者读取这些消息，就可能会破坏一致性。试想，一个消费者从当前Leader也就是副本0读取并处理了Message4，这个时候Leader挂了，选举了副本1作为新的Leader，此时另一个消费者再去从新的Leader读取消息，发现这个消息其实并不存在，这就导致了数据不一致性问题。

​		引入High Water Mark机制，会导致Broker间的消息赋值因为某些原因变慢，那么消息达到消费者的时间也会随之边长，因为我们会先等待消息复制完毕。延迟时间可以通过参数replica.lag.time.max.ms参数配置，它指定了副本在复制消息时可被允许的最大延迟时间。

# 6、ISR、OSR、AR是什么？

- ISR：In-Sync Replicas副本同步队列

- OSR：Out-of-Sync Replicas

- AR：Assigned Replicas所有副本

- ISR由leader维护，follower从leader同步数据有一些延迟，超过相应的阈值会把follower剔除ISR，存入到OSR列表，新加入的follower也会先存放在OSR中。AR=ISR+OSR。

# 7、LEO、HW、LSO、LW等分别代表什么？

- LEO：是LogEndOffset的简称，代表当前日志文件中最后一条
- HW：水位或者水印watermark，也叫作高水位High Watermark。通常被用在流式处理领域，比如Flink，Spark等。用来表示元素或者事件在基于事件层面上的进度。在Kafka中，水位的概念反而与时间无关，而是和位置信息有关。严格而说，它表示的位置信息，也就是位移offset，取partition对应的ISR中最小的LEO为HW，Consumer最多只能消费到HW所在的位置上一条消息。
- LSO：是LastStableOffset简称，对未完成的事务而言，LSO的值等于事务中第一条消息的位置firstUnstableOffset，对已完成的事务而言，它的值和HW相同。
- LW：Low Watermark低水位，代表AR集合中最小的logStartOffset值。
