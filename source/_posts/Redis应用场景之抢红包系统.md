---
title: Redis应用场景之抢红包系统(一)
date: 2018.06.19 10:06
tags:
  - 分布式中间件
description: Redis应用场景之抢红包系统(一)
toc: true
copyright: true
---

# 一、Redis应用场景之抢红包系统(一)

### 1、业务流程

- 信息流：包括用户操作背后的请求通信和红包信息在不同用户与用户群中的流转等
- 业务流：主要包括发红包、点红包和抢红包等业务逻辑
- 资金流：主要包括红包背后的资金转账和入账等流程

#### 1.1、业务系统流程图

![](Redis应用场景之抢红包系统/抢红包系统整体业务流程.jpg)

#### 1.2、业务流程分析

​		系统整体业务流程包括2大业务组成：发红包和抢红包。其中的拆红包又可以拆分2个小业务，用户点红包和用户拆红包。

##### 1.2.1、发红包整体业务流程

![](Redis应用场景之抢红包系统/发红包整体业务流程.jpg)

##### 1.2.2、抢红包整体业务流程

![](Redis应用场景之抢红包系统/抢红包整体业务流程.jpg)

### 2、业务模块划分

![](Redis应用场景之抢红包系统/业务模块划分.jpg)

- 缓存中间件Redis模块：主要用来发红包时缓存红包个数和由随机数算法产生的红包随机金额列表，同时将借助Redis单线程特性和操作的原子性实现抢红包时锁的操作
- 引入Redis一方面是大大减少高并发情况下频繁查询数据库的操作，减少数据库的压力。
- 另一方面是提供系统整体响应性能和保证数据的一致性。

### 3、数据库表设计与环境搭建

主要有3张表，发红包记录红包的信息表，发红包时对应的随机金额信息表，抢红包时用户抢到红包的金额记录表

#### 3.1、实体类Model

- 发红包记录表RedRecord

  ```java
  package com.victor.model.redpacket;
  
  import com.fasterxml.jackson.annotation.JsonFormat;
  import lombok.Data;
  import lombok.ToString;
  
  import javax.persistence.*;
  import java.math.BigDecimal;
  import java.util.Date;
  
  /**
   * @Description: 发红包记录表
    * @Author: VictorDan
   * @Version: 1.0
   */
  @Data
  @ToString
  @Entity
  @Table(name = "red_record")
  public class RedRecord {
      @Id
      @GeneratedValue(strategy = GenerationType.IDENTITY)
      @Column(name = "id")
      private Integer id;
      /**
       * 用户id
       */
      private Integer userId;
      /**
       * 红包全局唯一标识串
       */
      private String redPacket;
      /**
       * 红包指定可以抢的总人数
       */
      private Integer total;
      /**
       * 红包总金额
       */
      private BigDecimal amount;
      /**
       * 是否有效
       */
      private Boolean isActive;
  
      @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss",timezone = "GMT+8")
      private Date createTime;
  }
  
  ```

- 红包明细金额表RedDetail

  ```java
  package com.victor.model.redpacket;
  
  import com.fasterxml.jackson.annotation.JsonFormat;
  import lombok.Data;
  import lombok.ToString;
  
  import javax.persistence.*;
  import java.math.BigDecimal;
  import java.util.Date;
  
  /**
   * @Description: 红包明细金额表
   * @Author: VictorDan
   * @Version: 1.0
   */
  @Data
  @ToString
  @Entity
  @Table(name = "red_detail")
  public class RedDetail {
      @Id
      @GeneratedValue(strategy = GenerationType.IDENTITY)
      @Column(name = "id")
      private Integer id;
      /**
       * 红包记录id
       */
      private Integer recordId;
      /**
       * 红包随机金额
       */
      private BigDecimal amount;
      /**
       * 是否有效
       */
      private Boolean isActive;
  
      @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss",timezone = "GMT+8")
      private Date createTime;
  }
  ```

- 抢红包记录表RedRobRecord

  ```java
  package com.victor.model.redpacket;
  
  import com.fasterxml.jackson.annotation.JsonFormat;
  import lombok.Data;
  import lombok.ToString;
  
  import javax.persistence.*;
  import java.math.BigDecimal;
  import java.util.Date;
  
  /**
   * @Description: 抢红包记录表
   * @Author: VictorDan
   * @Version: 1.0
   */
  @Data
  @ToString
  @Entity
  @Table(name = "red_record")
  public class RedRobRecord {
      @Id
      @GeneratedValue(strategy = GenerationType.IDENTITY)
      @Column(name = "id")
      private Integer id;
      /**
       * 用户id
       */
      private Integer userId;
      /**
       * 红包全局唯一标识串
       */
      private String redPacket;
      /**
       * 抢到红包的金额
       */
      private BigDecimal amount;
      /**
       * 是否有效
       */
      private Boolean isActive;
      /**
       * 抢到时间
       */
      @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss",timezone = "GMT+8")
      private Date robTime;
  }
  ```

#### 3.2、持久层Repository

- 发红包记录表RedRecordRepo

  ```java
  package com.victor.repository.redpacket;
  
  import com.victor.model.redpacket.RedRecord;
  import org.springframework.data.jpa.repository.JpaRepository;
  import org.springframework.stereotype.Repository;
  
  /**
   * @Description:
   * @Author: VictorDan
   * @Version: 1.0
   */
  @Repository
  public interface RedRecordRepo extends JpaRepository<RedRecord,Long> {
  }
  ```

- 红包明细金额RedDetailRepo

  ```java
  package com.victor.repository.redpacket;
  
  import com.victor.model.redpacket.RedDetail;
  import org.springframework.data.jpa.repository.JpaRepository;
  import org.springframework.stereotype.Repository;
  
  /**
   * @Description:
   * @Author: VictorDan
   * @Version: 1.0
   */
  @Repository
  public interface RedDetailRepo extends JpaRepository<RedDetail,Long> {
  }
  ```

- 抢红包记录表RedRobRecordRepo

  ```java
  package com.victor.repository.redpacket;
  
  import com.victor.model.redpacket.RedRobRecord;
  import org.springframework.data.jpa.repository.JpaRepository;
  import org.springframework.stereotype.Repository;
  
  /**
   * @Description:
   * @Author: VictorDan
   * @Version: 1.0
   */
  @Repository
  public interface RedRobRecordRepo extends JpaRepository<RedRobRecord,Long> {
  }
  ```
