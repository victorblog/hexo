---
title: Java的常识总结
date: 2017.07.03 10:06
tags:
  - Java
description: Java的常识总结
toc: true
copyright: true
---

## 一、Java的常识总结

#### 1、final、finally、finalize的区别

- final：修饰类、方法、变量
  - 修饰类：类不能被其他类继承
  - 修饰方法：方法锁定，防止其他继承类进行更改
  - 修饰变量：表示为常量，只能被赋值一次，不能更改，而且必须要初始化。

- finally：只有与finally对应的try语句块得到执行的情况下，finally语句块才能执行。如果在执行try之前，方法发生异常，则finally不会执行。

- finalize：是java.lang.Object定义的，每一个方法都有finalize方法

  方法是在gc启动的时候，被调用。凡是new出来的对象，gc都能搞定，一般情况下我们又不会用new以外的方式去创建对象，所以一般是不需要程序员去实现finalize。

#### 2、HashMap与Hashtable

- 线程是否安全：HashMap非线程安全，Hashtable线程安全。Hashtable的方法内部都是由synchronized修饰。保证线程安全建议使用ConcurrentHashMap
- HashMap可以用null作为一个key键，但是只有一个，value为null可以有一个或者多个。<font color=red>Hashtable如果put进去的key为null就会报空指针。</font>
- HashMap默认的初始化为16，每次扩容会变为原来的2倍。JDK1.8以后的HashMap解决哈希冲突的时候，链表的长度默认值为8，<font color=red>如果超过8的时候，链表会转化为红黑树，用来减少搜索时间</font>。

#### 3、HashMap和HashSet

​	HashSet底层是基于HashMap实现。

| HashMap       | HashSet       |
| ------------- | ------------- |
| 实现Map接口   | 实现Set接口   |
| 存key-value   | 存Object      |
| 调用put()添加 | 调用add()添加 |

HashSet检查重复：当add的时候，HashSet会先计算对象的hashcode值来判断对象加入的位置。

也即是先判断hashcode值，如果hashcode相同，则调用equals()方法，如果为true，则相同,HashsSet就不会加入操作成功。

- HashMap多线程操作导致死循环问题，因为<font color=red>多线程并发情况下Rehash会造成元素之间形成一个循环链表。</font>

#### 4、ConcurrentHashMap

JDK1.8的ConcurrentHashMap取消了Segment分段锁，采用CAS和synchronized来保证并发安全。数据结构和1.8的HashMap类似，数组+链表/红黑二叉树。

当链表长度超过8，链表（查找复杂度O(N)）将转换为红黑树（查找复杂度O(longN)）

synchronized只锁定当前链表或者红黑二叉树的首节点，这样只要hash不冲突，就不会产生并发，效率又提升了N倍。

#### 5、线程

线程是一个比进程更小的执行单位，一个进程在其执行的过程中可以产生多个线程。

与进程不同的是，类的多个线程共享进程的堆Heap和方法区Method Area资源。

每个线程都有自己的程序计数器PC，虚拟机栈Stack，本地方法栈Native Method Stack。

一个进程中可以有多个线程，多个线程共享进程的堆和方法区（JDK1.8以后的没有方法区，从JVM移到本地成为元空间MetaSpace）

- 程序计数器PC：字节码解释器通过改变程序计数器来读取指令，主要是为了线程切换后能恢复到正确的执行位置。

- 虚拟机栈和本地方法栈
  - 虚拟机栈：每个java方法在执行的同时会创建一个栈帧用来存储局部变量表，操作数栈，常量池引用的信息。
  - 本地方法栈：虚拟机栈是为Java方法也就是字节码服务，本地方法栈为虚拟机用到的Native方法服务。

- 堆和方法区

  是所有线程共享的资源，堆是进程中最大的一块内存，主要用来存放新创建的对象，所有对象都在这里分配内存，用来存放新创建的对象。

  方法区：用来存放已被加载的类信息，常量，静态变量。

#### 6、sleep()方法和wait()方法

wait()方法是java.lang.Object里的方法，用来线程间交互/通信

sleep()是thread的方法。sleep方法没有释放锁，wait会释放锁，导致其他进程占用锁。

wait()方法被调用后，线程不会自动苏醒，需要别的线程调用同一个对象上的notify()或者notifyAll()方法。

sleep()方法执行完后，线程会自动苏醒。

#### 7、start()方法和run()方法

Thread thread=new Thread();线程就进入了新建装填，调用了start()方法，就会启动一个线程进入就绪状态。当分配到时间片就可以运行了。

start()：执行线程相应的准备工作，然后自动执行run()方法的内容。

直接执行run()，会把run()方法当成一个main主线程的普通方法去执行，并不会再某个线程中执行它。

run方法只是thread的一个普通方法调用，还是在主线程里执行。

#### 8、synchronized关键字

synchronized关键字可以保证被修饰的方法、代码块在任何时刻都只有一个线程执行。

JDK1.6之前，synchronized属于重量锁，因为使用监视器monitor，它是依赖于操作系统Mutex lock。需要进行线程切换从用户态到和心态。

JDK1.6之后，引入了锁优化，自选锁，锁消除，锁粗化，偏向锁。等减少了锁操作的开销。

#### 9、synchronized和ReentrantLock

都是可重入锁：自己可以再次获取自己的内部锁，比如一个线程获得了某个对象的锁，此时这个对象锁还没有释放，当再次想要获取这个对象的锁还是可以获取的，如果不可重入，就会死锁。

ReentrantLock是JDK层面实现的，需要用lock()和unlock()方法配合try/finally语句块。

ReentrantLock增加的高级功能：

- 等待可中断：lock.lockInterruptibly();

- 可实现公平锁：ReentrantLock(boolean fair)://值为true，则为公平锁。

- 可实现选择性通知：需要借助Condition接口和newCondition()，来实现线程通信

  和wait()与notify()/notifyAll()原理类似。而Condition的singnalAll()方法会唤醒所有线程。

#### 10、volatile关键字

Java的内存模型总是从主存（共享内存）读取变量，保证内存可见性CAS。

防止指令重排序。

## 二、HTTP请求头

http请求头，http客户程序（浏览器），向服务器发送请求的时候必须指明请求类型（GET或者POST），如果必要还可以选择发送其他的请求头。

- Accept：浏览器可以接受的MIME类型
- Accept-Charset：浏览器可接受的字符集
- Accept-Encoding：浏览器能够进行解码的数据编码方式，比如gzip。Servlet能够向支持gzip的浏览器返回经过gzip编码的HTML页面，许多情形下可以减少5到10倍的下载时间
- Accept-Language：浏览器所希望的语言种类，当服务器能够提供一种以上的语言版本时使用。
- Authorization：授权信息，通常出现在服务器发送的token中
- Connection：表示是否需要持久连接，如果Servlet看到这里的值为Keep-Alive，或者请求使用的是Http1.1（默认为持久连接）
- Cookie：最重要的请求头信息之一
- Host：初始URL中的主机和端口
- Referer：包含一个URL，用户从该URL代表的页面出发访问当前请求的页面
- User-Agent：浏览器类型，如果Servlet返回的内容与浏览器类型有关则该值非常有用。