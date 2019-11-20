---
title: Java的锁分类
date: 2018.06.11 10:06
tags:
  - Java
description: Java的锁
toc: true
copyright: true
---

### 一、Java锁分类

#### 1、乐观锁

- 乐观锁是一种乐观思想，认为<font color=red>读多写少，遇到并发写的可能性低，每次拿数据的时候都认为别人不会修改，所以不会上锁</font>。

- 但是在更新的时候会判断一下在期间别人有没有去更新这个数据，采取在写时先读出当前版本号，然后加锁操作。比较跟上一次的版本号，如果一样则更新，如果失败则要重复读-比较-写的操作。

- Java的乐观锁基本都是通过CAS操作实现的。CAS是一种更新的原子操作，比较当前值是否一样，一样则更新，否则失败。

#### 2、悲观锁

- 悲观锁就是悲观思想，认为写多，遇到并发写的可能性高，每次去拿数据的时候都认为别人会修改，所以每次在读写数据的时候都会上锁，这样别人想读写这个数据就会block直到拿到锁。java中的悲观锁就是

- synchronized，AQS框架下的锁则是先尝试CAS乐观锁去获取锁，获取不到才会转换为悲观锁，比如RetreenLock。

#### 3、自旋锁

- 自选锁原理非常简单，如果持有锁的线程能在很短时间内释放锁资源，那么那些等待竞争锁的线程就不需要做内核态和用户态之间的切换进行阻塞挂起状态，他们只需要等一等（自旋），等持有锁的线程释放锁后就可以立即获取到锁，这样就避免了用户线程和内核的切换消耗。

- 线程自旋需要消耗CPU，说白了就是让CPU再做无用功，如果一直获取不到锁，那么线程也不能一直占用CPU自旋做无用功，所以需要设定一个自旋等待的最大时间。

- 如果持有锁的线程执行的时间超过自旋等待的最大时间仍没有释放锁，就会导致其他的线程在最大等待时间内还是获取不到锁，这时其他线程就会停止自旋进入阻塞状态。

- <font color=red>自旋锁是一种思想，一般需要配合CAS使用</font>

- <font color=red>java.util.concurrent.atomic包下的原子类是自旋锁</font>

#### 4、可重入锁

- 不可重入的话，一个锁在嵌套中使用会把自己锁死
- synchronized和ReentrantLock都是可重入锁，可以放心用

#### 5公平锁/非公平所

- synchronized是非公平锁，ReentrantLock默认构造函数也是非公平锁
- 非公平锁的性能比公平锁性能高很多

#### 6、互斥锁/共享锁

- 互斥与共享的概念简单，任何语言都存在

#### 7、偏向锁/轻量级锁/重量级锁

- synchronized底层优化，偏向锁、轻量级锁是针对重量级锁做优化而提出来的定义
- 这些优化大部分情况下对于开发来讲是透明的，默认开启的

#### 8、分段锁

- 分段锁不是一种实际的锁，而是一种思想
- ConcurrentHashMap是实现锁分段机制

#### 9、锁优化

##### 9.1、减少锁持有时间

只用在要求线程安全的程序上加锁

##### 9.2、减小锁粒度

将大对象(被多线程访问的对象)，拆分成小对象，大大增加并行度，降低锁竞争。降低锁竞争，偏向锁，轻量级锁成功率才会提高。最典型的减小锁粒度的例子就是：分段锁ConcurrentHashMap。

#### 10、锁分离

- <font color=red> 常见的锁分离就是读写锁ReadWriteLock</font>，根据功能进行分离成读锁和写锁，这样读读不互斥，读写互斥，写写互斥，保证了线程安全，还提高了性能。
- LinkedBlockingQueue从头部取出数据，从尾部放数据。

#### 11、锁粗化

为了保证多线程间的有效并发，会要求每个线程持有锁的时间尽量短，也就是在使用公共资源后，应该立即释放锁。<font color=red>如果对同一个锁不停地进行请求，同步和释放，本身也会消耗系统的资源，反而不利于性能的优化</font>

#### 12、锁消除

锁消除是在编译器级别的事情，如果发现不可能被共享的对象，就可以消除这些对象的加锁操作，多数是因为编码不规范引起的。

### 二、Java常用锁

- synchronized
- ReentrantLock
- ReadWriteLock
- Semaphore

#### synchronized

- 按照加锁范围大小，分为类锁和对象锁
- 按加锁方法，分为代码块加锁和方法加锁

注：<font color=red>对象锁只会影响单个对象，类锁会影响这个类下的所有对象</font>

##### 1、分析

- 每个对象都有一个monitor对象，加锁就是在竞争monitor对象
- 代码块加锁是在前后分别加上monitorenter和monitorexit指令来实现的
- 方法加锁是通过一个标记位来判断的

##### 2、误解

- JDK1.5，synchronized是一个重量级加锁操作，需要调用操作系统的接口，则导致性能低，给线程加锁消耗的时间比有用操作消耗时间要多。

- JDK1.6，synchronized进行了锁优化，有自旋锁，锁消除 ，锁粗化，轻量级锁，偏向锁等，效率有了提高。而之后的JDK1.7,1.8都对关键字的实现原理实现了优化。

##### 3、总结

- 引入了偏向锁和轻量级锁，都是在对象头中有标记位，不需要经过操作系统加锁
- 锁可以从偏向锁升级到轻量级锁，再升级到重量级锁。这种升级过程叫做锁膨胀
- JDK1.6中默认是开启偏向锁和轻量级锁，可以通过-XX:UseraBiasedLocking来禁用偏向锁

#### ReentrantLock

##### 1、使用

- lock.lock()和lock.unlock()

``` java
package lock;

import java.util.concurrent.locks.ReentrantLock;

/**
 * @Description:
 * @Author: VictorDan
 * @Date: 18-11-20 下午4:15
 * @Version: 1.0
 */
public class LockTest extends Thread {
    public static ReentrantLock lock = new ReentrantLock();
    public static int flag = 0;

    public LockTest(String name) {
        super(name);
    }

    @Override
    public void run() {
        for (int i = 0; i < 10000; i++) {
            try{
                lock.lock();
                flag++;
            }finally {
                lock.unlock();
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        LockTest lockTest1=new LockTest("thread1");
        LockTest lockTest2=new LockTest("thread1");
        lockTest1.start();
        lockTest2.start();
        lockTest1.join();
        lockTest2.join();
        /**
         * 如果不加锁，则flag为小于20000的任意数
         */
        System.out.println(flag);//2000
    }
}
```

- lock.tryLock(long timeout, TimeUnit unit)

  尝试获取锁，在时间范围内没有拿到就会返回false，不会永久构成死锁。

```java
package com.anjuke.ai;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.ReentrantLock;

/**
 * @Description:
 * @Author: VictorDan
 * @Date: 18-11-20 下午4:24
 * @Version: 1.0
 */
public class TryLock extends Thread {
    public static ReentrantLock lock = new ReentrantLock();

    public TryLock(String name){
        super(name);
    }

    @Override
    public void run() {
        try {
            if (lock.tryLock(5, TimeUnit.SECONDS)) {
                Thread.sleep(6000);
            } else {
                System.out.println(this.getName() + " get lock failed");
            }
        } catch (Exception e) {
        } finally {
            if (lock.isHeldByCurrentThread()) {
                System.out.println("lock.isHeldByCurrentThread: " + this.getName());
                lock.unlock();
            }
        }
    }

    public static void main(String[] args) {
        TryLock t1 = new TryLock("TryLockTest1");
        TryLock t2 = new TryLock("TryLockTest2");

        t1.start();
        t2.start();
    }
}

//TryLockTest2 get lock failed
//lock.isHeldByCurrentThread: TryLockTest1
```

- 公平锁：
  - 一般锁是不公平的，不一定先来的线程先得到锁，后来的锁就后得到锁，不公平锁可能会产生饥饿现象
  - 公平锁就是先来先服务。不会产生饥饿现象，但是公平所性能比费公平所性能差很多

``` java
public static ReentrantLock lock = new ReentrantLock(true);
```

##### 2、分析

- ReentrantLock是基于AQS实现的，而AQS的基础是CAS。所以搞定AQS，就搞定了ReentrantLock。

- ReentrantLock分为公平锁和非公平锁
- 而ReentrantLock，CountDownLatch，Semaphore都是通过AQS实现的

##### 3、总结

- synchronized能做的，ReentrantLock都能做，并且还能做很多，但是synchronized仍有用武之地
- ReentrantLock相比synchronized的优势是可中断，公平锁，多个锁。这种情况下使用ReentrantLock。只要是synchronized能做到的，还是使用synchronized