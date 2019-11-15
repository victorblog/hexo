---
title: Java编码规范
date: 2018.02.01 10:06
tags:
  - Java
description: Java编码规范
toc: true
copyright: true
---

## 一、Java编码规范

### 1、命名风格

1. 代码中的命名不能以_，$开始和结束

2. 代码中的命名严禁使用拼音与英文混合，不允许使用中文的方式

3. 常量命名全部大写，单词之间下划线隔开。

4. POJO类中布尔类型，都不要加is，否则框架解析出现序列化错误。

5. 对于Service和DAO类，基于SOA理念，暴露出服务一定是接口。

   内部的实现类用Impl。比如CacheServiceImpl是CacheService接口。

6. 枚举类名要加上Enum后缀，枚举成员名称要全部大写，单词之间有下划线隔开。

   ```java
   public enum DealStatusEnum{
       SUCCESS,UNKOWN_REASON
   }
   ```

7. 如果用到了设计模式，建议在类名中体现出具体设计模式

   ```java
   public class LoginProxy
   ```


### 2、代码风格

1. 左右小括号和字符之间不能有空格

2. 单行字符数限制不超过120个，超出需要换行。

   ```java
   sb.append("zi").append("xin")
       .append("victor")
       .append("victor")
   ```

3. 方法体内的执行语句组、变量的定义语句组、不同的业务逻辑之前或者不同的语义之间插入一个空行。

   相同的业务逻辑和语义之间不需要插入一个空行

4. if/for/while/switch/do等括号之间必须加空格

5. 方法参数在定义和传入时，多个参数逗号后边必须加空格

   ```java
   method("a", "b", "c");
   ```

### 3、OOP规范

1. 所有的重写方法，必须加上@Override注解；注：是为了让编译器帮你发现错误

2. 相同参数类型，相同业务含义，才可以使用java的可变参数，避免使用Object可变参数。

3. Object的equals方法容易抛出空指针异常，应使用常量或者确定有值得对象来调用equals

   ```java
   "test".equals(object);
   ```

4. 所有的POJO类属性必须使用包装数据类型；RPC方法的返回值和参数必须使用包装数据类型

   注意：为了避免在POJO转换过程中失败。

5. 定义DO/DTO/VO等POJO类时，不要设定任何属性默认值。

   注意：因为POJO原则转换过程中避免失败。

6. 类内方法定义顺序依次是：public方法/protected方法>>private方法>>getter/setter方法

   注意：把最关注的方法放在前面

7. getter/setter方法中，不要增加业务逻辑，增加排查问题的难度

8. 慎用Object的clone方法来拷贝对象，深度复制需要自己实现。

9. 循环体内，字符串的连接方式，使用StringBuilder的append方法进行扩展

10. 拒绝巨型方法，把方法拆解到尽可能符合单一责任原则的粒度，方便维护及复用。

### 4、集合处理

1. 只要重写equals，就必须重写hashCode()，Set，Map对象作为键的情况小重写hashCode,equals
2. ArrayList的subList结果不可强转成ArrayList，否则会抛出ClassCastException强制类型转换异常。
3. 使用工具类Arrays.asList()把数组转换为集合时，不能使用其修改集合相关的方法，它的add/remove/clear方法会抛出UnsupportedOperationException异常。


