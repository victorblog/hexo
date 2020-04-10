---
title: Redis的基本使用
date: 2018.05.11 10:06
tags:
  - 分布式中间件
description: Redis的基本使用
toc: true
copyright: true
---

# 一、Redis的基本使用总结
### 1、RedisTemplate与StringRedisTemplate的使用
- 两个操作组件都可以用来操作存储字符串类型数据信息
- 对于RedisTemplate，还可以用来操作List，Set，SortedSet，Hash等
### 2、Redis的List列表
- 和Java的List类型很类似，用来存储一系列具有相同类型的数据，底层对于数据的存储和读取可以理解为
一个数据队列，往List中添加数据，相当于从队列中的队尾插入，获取数据从队头获取数据。
- 总结：使用List存入对象的时候，JavaBean必须要实现Serializable接口序列化，因为Spring会将对象先序列化后再存入Redis，否则会报错。
### 3、Redis的Set集合
- Redis的Set和Java里的Set集合类似性质，存储的数据相同类型且不重复，也就是Redis集合中的数据是唯一的，底层的数据结构是通过Hash表来实现的。所以对它增、删、查操作的复杂度都是O(1)。
- 使用场景：在实际开发中，set常用来解决重复提交，剔除重复id等业务场景
### 4、Redis的有序集合Zset
- Redis的Zset和Set具有某些相同特征，也就是存储的数据不重复，不同之处
在Zset可以通过底层的Score(分数/权重)值对数据进行排序，实现存储的集合数据有序不重合。
- 使用场景：Zset常用来充值排行榜，积分排行榜，成绩排名等应用场景。
### 5、Redis的Hash存储
- Redis的Hash有点类似Java的HashMap，底层数据结构是Key-Value的哈希表。
- 使用场景：当需要存入redis中的对象信息具有某种共性，为了减少缓存中key的数量，应考虑采用hash存储。
### 6、使用Redis总结
- 如果已经用Redis将JavaBean保存到集合类型中，如果后续再修改JavaBean的类型，重新插入，会报错
local class incompatible: stream classdesc serialVersionUID = 72892319061181, local class serialVersionUID = -3998150864330771094
- 出现这个问题后，不论是在model类里面添加
  private static final long serialVersionUID = -6743567631108323096L;
  还是重新Build都还是有这些问题。是因为第一次保存数据到Redis进入需要进行序列化，而修改JavaBean的属性类型，会导致序列化的UID不一样，自然无法操作。如果想要重新插入数，需要把所有的对应的key清空
  把redis服务关闭或者清空后再登录就好了。
- 因为redis存的是字节数据，所以model必须要序列化，改变model的属性值类型，序列化UID会发生变化。
### 7、Redis中Key的失效与判断是否存在
- 在有些业务场景下，redis的key对应的数据不需要永久保留，这个时候就需要对缓存中的key进行清理。
- redis缓存架构中，delete与expire操作都可以用来清理key。
- 区别：
    - delete操作是人为手动触发，
    - expire只需要提供一个ttl过期时间，key就自动失效，会被自动清理
#### 7.1、在调用SETEX方法的时候，指定key的过期时间
- 方法1：valueOperations.set(key,"expire操作",10L, TimeUnit.SECONDS);
- 方法2：valueOperations.set(key,"expire操作");
              redisTemplate.expire(key,10L,TimeUnit.SECONDS); 
#### 7.2、判断key是否存在
- 使用redisTemplate.hasKey(key)判断key是否存在

# 二、Redis使用的实例代码

```java
package com.victor.redis;

import com.fasterxml.jackson.databind.ObjectMapper;
import com.victor.AbstractTest;
import com.victor.model.Account;
import com.victor.model.User;
import lombok.extern.slf4j.Slf4j;
import org.junit.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.*;

import java.util.ArrayList;
import java.util.List;
import java.util.Map;
import java.util.Set;
import java.util.concurrent.TimeUnit;

@Slf4j
public class RedisTest extends AbstractTest {
    /**
     * 由于之前已经自定义注入RedisTemplate组件，所以可以直接自动装配
     */
    @Autowired
    private RedisTemplate redisTemplate;

    /**
     * redis的set简单测试
     */
    @Test
    public void getDataFromRedis() {
        log.info("-------------开始RedisTemplate操作组件实战-------------");
        final String content = "你淡爷爷";
        final String key = "redis:template:victor:string";
        //redis通用的操作组件
        ValueOperations valueOperations = redisTemplate.opsForValue();
        log.info("写入redis中的内容：{}", content);
        //将字符串信息写入到redis中
        valueOperations.set(key, content);
        Object result = valueOperations.get(key);
        log.info("从redis中读取的内容：{}", result);
    }

    /**
     * json的序列化与反序列化框架
     */
    @Autowired
    private ObjectMapper objectMapper;

    @Test
    public void testUser() {
        log.info("-------------开始RedisTemplate操作组件实战-------------");
        User user = new User(1, "victor", "你淡爷爷");
        ValueOperations valueOperations = redisTemplate.opsForValue();
        final String key = "redis:template:victor:object";
        String content;
        try {
            content = objectMapper.writeValueAsString(user);
            valueOperations.set(key, content);
            log.info("写入redis中的对象信息为：{}", user);
            Object result = valueOperations.get(key);
            if (result != null) {
                User value = objectMapper.readValue(result.toString(), User.class);
                log.info("从redis中读取内容并反序列化的结果为：{}", value);
            }
        } catch (Exception e) {
            log.error("json序列化失败：{}", e.getMessage());
        }

    }

    /**
     * RedisTemplate的特列，专门用来处理缓存中value的数据类型为String类型。
     */
    @Autowired
    private StringRedisTemplate stringRedisTemplate;

    /**
     * StringRedisTemplate的简单测试
     */
    @Test
    public void testStringRedisTemplate() {
        log.info("-------------开始StringRedisTemplate操作组件实战-------------");
        final String content = "StringRedisTemplate的使用";
        final String key = "redis:string";
        ValueOperations<String, String> valueOperations = stringRedisTemplate.opsForValue();
        log.info("写入redis的内容：{}", content);
        valueOperations.set(key, content);
        String result = valueOperations.get(key);
        log.info("从redis中获取的内容为：{}", result);
    }

    /**
     * 采用StringRedisTemplate将对象信息系列化为json格式字符串后写入redis
     * 然后从redis中读取出来，最后反序列化
     */
    @Test
    public void testStringRedisTemplateObject() {
        log.info("-------------开始StringRedisTemplate操作组件实战-------------");
        User user = new User(1, "victor", "你淡爷爷");
        ValueOperations<String, String> valueOperations = stringRedisTemplate.opsForValue();
        final String key = "redis:string:object";
        String content;
        try {
            content = objectMapper.writeValueAsString(user);
            valueOperations.set(key, content);
            log.info("写入redis中的对象信息为：{}", user);
            Object result = valueOperations.get(key);
            if (result != null) {
                User value = objectMapper.readValue(result.toString(), User.class);
                log.info("从redis中读取内容并反序列化的结果为：{}", value);
            }
        } catch (Exception e) {
            log.error("json序列化失败：{}", e.getMessage());
        }
    }

    /**
     * Redis中的List的测试使用
     */
    @Test
    public void testRedisList() {
        log.info("-----------Redis中的List数据类型测试使用-----------------");
        List<User> list = new ArrayList<>();
        for (int i = 0; i < 10; i++) {
            User user = new User(i, i + "号男嘉宾", "张" + i);
            list.add(user);
        }
        log.info("构造已经排好序的User的列表为：{}", list);
        final String key = "redis:list";
        ListOperations listOperations = redisTemplate.opsForList();
        for (User user : list) {
            //往redis的list中添加数据--->从list队尾添加
            listOperations.leftPush(key, user);
        }
        log.info("---------获取Redis中List的数据--->从队尾中获取----------");
        Object data = listOperations.rightPop(key);
        User user;
        while (data != null) {
            user = (User) data;
            log.info("当前获取到的数据为：{}", user);
            data = listOperations.rightPop(key);
        }
    }

    /**
     * Redis的Set的测试使用
     */
    @Test
    public void testRedisSet() {
        log.info("-----------Redis中的Set数据类型测试使用-----------------");
        List<String> list = new ArrayList<>();
        list.add("hello");
        list.add("victor");
        list.add("你淡爷爷");
        list.add("redis");
        list.add("list");
        list.add("set");
        list.add("hello");
        list.add("victor");
        log.info("待处理的用户姓名列表：{}", list);
        final String key = "redis:set";
        SetOperations setOperations = redisTemplate.opsForSet();
        /**
         * 遍历访问list，并剔除相同的名字存入的Redis的set
         */
        for (String str : list) {
            setOperations.add(key, str);
        }
        //从Redis的set中获取对象的集合
        Object result = setOperations.pop(key);
        while (result != null) {
            log.info("当前获取到的数据为：{}", result);
            result = setOperations.pop(key);
        }
    }

    /**
     * Redis的zset的操作使用
     */
    @Test
    public void testRedisZset() {
        log.info("-----------Redis中的Zset数据类型测试使用-----------------");
        List<Account> list = new ArrayList<>();
        for (int i = 101; i < 108; i++) {
            Account account = new Account(i + "", i + Math.random());
            list.add(account);
        }
        log.info("待处理的用户姓名列表：{}", list);
        final String key = "redis:zset";
        //获取有序集合zset的操作组件
        ZSetOperations zSetOperations = redisTemplate.opsForZSet();
        /**
         * 遍历访问list，并剔除相同的名字存入的Redis的set
         */
        for (Account account : list) {
            zSetOperations.add(key, account, account.getMoney());
        }
        //从Redis的zset中获取对象的集合的size大小
        Long size = zSetOperations.size(key);
        log.info("-------------从小到大排名--------------");
        Set<Account> range = zSetOperations.range(key, 0L, size);
        for (Account data : range) {
            log.info("从redis中读取money排序列表，当前获取到的数据为：{}", data);
        }
        log.info("-------------从大到小排名--------------");
        Set<Account> reverseRange = zSetOperations.reverseRange(key, 0L, size);
        for (Account data : reverseRange) {
            log.info("从redis中读取money排序列表，当前获取到的数据为：{}", data);
        }
    }

    /**
     * Redis的Hash测试使用
     */
    @Test
    public void testRedisHash() {
        log.info("-----------Redis中的Hash数据类型测试使用-----------------");
        List<User> userList = new ArrayList<>();
        List<Account> accountList = new ArrayList<>();
        for (int i = 100010; i < 100016; i++) {
            User user = new User(i, "张三" + i, "user" + i);
            userList.add(user);
        }
        for (int i = 101; i < 106; i++) {
            Account account = new Account(i + "", i + Math.random());
            accountList.add(account);
        }
        final String userKey = "redis:hash:user";
        final String accountKey = "redis:hash:account";
        //获取Hash存储的操作组件
        HashOperations hashOperations = redisTemplate.opsForHash();
        /**
         * 分别遍历userList和accountList加入的redis
         */
        for (User user : userList) {
            hashOperations.put(userKey, user.getId().toString(), user);
        }
        for (Account account : accountList) {
            hashOperations.put(accountKey, account.getUserId(), account);
        }
        /**
         * 获取user对象列表和account对象列表
         */
        Map<String, User> userMap = hashOperations.entries(userKey);
        log.info("获取userList的数据：{}", userMap);
        Map<String, Account> accountMap = hashOperations.entries(accountKey);
        log.info("获取accountList的数据：{}", accountMap);
        //获取指定的user对象
        String userId = "100010";
        User user = (User) hashOperations.get(userKey, userId);
        log.info("获取指定的user对象：{}->{}", userId, user);
        //获取指定的account对象
        String accountId = "101";
        Account account = (Account) hashOperations.get(accountKey, accountId);
        log.info("获取指定的user对象：{}->{}", accountId, account);
    }

    /**
     * redis中存数据的时候，给key设置过期测试使用
     * @throws Exception
     */
    @Test
    public void testRedisKeyExpire1() throws Exception{
        final String key="redis:expire:key";
        ValueOperations valueOperations = redisTemplate.opsForValue();
        /**
         * 方法1：在往redis中set数据的时候，提供一个TTL，ttl时间一到，redis中的key自动失效，会被清理
         */
        valueOperations.set(key,"expire操作",10L, TimeUnit.SECONDS);
        /**
         * 等待5秒判断key是否还存在
         */
        Thread.sleep(5000);
        Boolean hasKey = redisTemplate.hasKey(key);
        Object value = valueOperations.get(key);
        log.info("等待5秒---->判断key是否存在：{}，对应的值为：{}",hasKey,value);
        /**
         * 再等待5秒判断key是还存在
         */
        Thread.sleep(5000);
        hasKey = redisTemplate.hasKey(key);
        value = valueOperations.get(key);
        log.info("再等待5秒---->判断key是否存在：{}，对应的值为：{}",hasKey,value);
    }

    /**
     * redis中存数据的时候，给key设置过期测试使用
     * 使用redisTemplate的expire方法
     * @throws Exception
     */
    @Test
    public void testRedisKeyExpire2() throws Exception{
        final String key="redis:expire:key";
        ValueOperations valueOperations = redisTemplate.opsForValue();
        /**
         * 方法2：在往redis中set数据的时候，使用redisTemplate的expire方法给key设置过期时间
         */
        valueOperations.set(key,"expire操作");
        redisTemplate.expire(key,10L,TimeUnit.SECONDS);
        /**
         * 等待5秒判断key是否还存在
         */
        Thread.sleep(5000);
        Boolean hasKey = redisTemplate.hasKey(key);
        Object value = valueOperations.get(key);
        log.info("等待5秒---->判断key是否存在：{}，对应的值为：{}",hasKey,value);
        /**
         * 再等待5秒判断key是还存在
         */
        Thread.sleep(5000);
        hasKey = redisTemplate.hasKey(key);
        value = valueOperations.get(key);
        log.info("再等待5秒---->判断key是否存在：{}，对应的值为：{}",hasKey,value);
    }
}

```

