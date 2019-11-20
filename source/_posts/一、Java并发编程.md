---
title: Java并发编程
date: 2018.08.01 10:06
tags:
  - Java
description: Java并发编程
toc: true
copyright: true
---

# 一、Java并发编程

- 多线程程序包含两个或者多个可以同事运行的部分，每个部分可以同事处理不同的任务，从而能更好的利用可用资源，如果CPU是多核，多线程可以写入多个task，在同一个程序同时进行操作处理。

- 多任务是多个进程共享，比如CPU处理公共资源

- 多线程：将多任务的概念扩展到可以将单个应用程序中的特定操作细分为单个线程的应用程序 ，每个线程可以并行运行。

## 1、线程的生命周期

线程在生命周期中经历多个阶段：线程诞生，启动，运行，然后死亡。

- 新线程New：新线程在新的状态下开始其生命周期，直到程序启动线程为止，保持在这种状态，也叫出生线程。
- 可运行Runnable：新诞生的线程启动以后，start，该线层可以运行，状态的线程被认为正在执行其任务。
- 等待Waiting：有时线程回转换到等待状态sleep,wait()，而线程等待另一个线程执行任务。只有当另一个线程发信号通知notify()/notifyAll()等待线程才能继续执行时，线程才转回到可运行状态。
- 定时等待Timed Waiting：可运行的线程可以在指定的时间间隔内进入定时等待状态，当该时间间隔sleep(100)到期或者发生等待的事件的时候，此状态的线程将转换到可运行状态。
- 终止dead：可执行线程在完成任务或者以其他方式终止的时候进入到终止状态。

## 2、线程优先级

每个java线程都有一个优先级，可以帮助操作系统安排线程的顺序。Java线程优先级在MIN_PRIORITY(1),和MAX_PRIORITY(10)之间的范围内，默认情况下的，每个线程优先级都是NORM_PRIORITY(5)

具有较高的优先级的线程相对于一个程序来说更重要，应该在低优先级线程之前分配处理器时间。然而，线程优先级并不能呢保证线程执行的顺序。

## 3、通过实现Runnable接口创建线程

- 需要实现Runnable接口提供的run()方法。run()方法为线程提供一个入口，可以把完整的业务逻辑放到run()方法体。此方法是没有返回值的

  ```java
  public void run(){
      ...service....
  }
  ```

- 然后在通过Thread构造函数，去创建一个对象
- 对象调用start()方法来启动线程。

```java
package multi_thread;

/**
 * @Description:通过实现Runnable接口的方式创建线程
 * 1、首先是需要实现Runnable的接口的run方法，run()作为线程提供一个入口，完成的业务逻辑写在run()里
 * 2、通过Thread构造函数，去创建一个实例对象
 * 3、对象调用start()方法来启动线程
 * @Author: VictorDan
 * @Date: 19-8-12 下午3:58
 * @Version: 1.0
 */
public class TestRunnable implements Runnable {
    private Thread thread;
    private String threadName;

    public TestRunnable(String threadName) {
        this.threadName = threadName;
        System.out.println("创建线程："+threadName);
    }

    /**
     * 实现Runnable接口，需要重写run方法
     * run方法里一般写的业务逻辑代码
     */
    @Override
    public void run() {
        System.out.println("创建线程："+threadName);
        try {
            for (int i = 0; i < 5; i++) {
                System.out.println("线程："+threadName+","+i);
                Thread.sleep(50);
            }
        }catch(InterruptedException e){
            System.out.println("线程："+threadName+"被打断");
        }
        System.out.println("线程:"+threadName+"执行完退出");
    }

    public void start(){
        System.out.println("开启线程："+threadName);
        if(thread==null){
            thread=new Thread(this,threadName);
            thread.start();
        }
    }

    public static void main(String[] args) {
        TestRunnable thread1 = new TestRunnable("Thread-1");
        thread1.start();
        TestRunnable thread2 = new TestRunnable("Thread-2");
        thread2.start();
    }
}

```

## 4、通过继承Thread类创建一个线程

## 5、线程池的使用

```java
    public static void main(String[] args) {
        /**
         * 使用Executors可以创建线程池，但是你方便的同时，隐藏了复杂性，而埋下隐患，比如OOM，线程耗尽等。
         * 所以阿里开发规范，创建线程池要手动创建，而不允许Executors直接调用的方式。
         */
        //1、创建一个不限制线程数上限的线程池，任何提交的任务都立即执行，最大上限是Integer.MAX_VALUE,会使用
        Executors.newCachedThreadPool();
        //2、创建一个固定个数大小的线程池
        Executors.newFixedThreadPool(10);
        //3、创建一个只有1个线程的线程池
        Executors.newSingleThreadExecutor();
        //4、创建一个定时的线程池，工作队列是延迟队列。
        Executors.newScheduledThreadPool(20);
        //总结：小程序使用这些快捷方法创建线程池没问题，但是对于服务端长期运行的程序，需要如下通过利用ThreadPoolExecutor的构造函数创建线程池。
        /**
         * Executors.newXXXThreadPool(),这种方式创建线程池，会使用无界的任务队列，为了避免OOM，应该手动使用ThreadPoolExecutor的构造方法指定队列的最大长度。
         * 还要明确拒绝任务时的策略，行为。也就是任务对了沾满的时候，这里在submit()新的任务，
         * public interface RejectedExecutionHandler{
         *     void rejectedExecution(Runnable r,ThreadPoolExecutor executor);
         * }
         * 线程池提供了常见的拒绝策略。
         * 1、new ThreadPoolExecutor.DiscardOldestPolicy():丢弃执行队列中最老的任务，尝试为当前提交的任务腾出位置
         * 2、new ThreadPoolExecutor.DiscardPolicy()：什么都不做，直接忽略
         * 3、new ThreadPoolExecutor.AbortPolicy();抛出RejectedExecutionException
         * 4、new ThreadPoolExecutor.CallerRunsPolicy();直接由提交任务者执行这个任务
         * 线程池默认的拒绝行为是：AbortPolicy，也就是抛出RejectedExecutionException，
         * 如果不关心任务被拒绝，则可以使用DiscardPolicy，这样多余的任务可以悄悄的被忽略。
         */
        //Executors的方法也是使用ThreadPoolExecutor的构造方法创建的线程池。
        new ThreadPoolExecutor(2, 20, 0L, SECONDS, new ArrayBlockingQueue<>(512), new ThreadPoolExecutor.DiscardOldestPolicy());
        ExecutorService executorService = Executors.newCachedThreadPool();
        new ThreadPoolExecutor.DiscardPolicy();
        new ThreadPoolExecutor.AbortPolicy();
        new ThreadPoolExecutor.CallerRunsPolicy();
        new ThreadPoolExecutor.DiscardOldestPolicy();
        //executorService=Executors.newSingleThreadExecutor();
        for (int i = 0; i < 20; i++) {
            executorService.execute(new WorkTask());
        }
        Future<Object> future = executorService.submit(() -> {
            //这个异常会在调用Future.get()的时候传递给调用者
            throw new RuntimeException("exception in call");
        });
        try {
            Object result = future.get();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            //exception in Callable.call();
            e.printStackTrace();
        } finally {
            executorService.shutdown();
        }

        /**
         * 线程池的常用场景
         * 1、如何正确的构造线程池
         * 2、构造一个工作阻塞队列
         * 3、设置拒绝策略
         * 设置
         */
        int poolSize = Runtime.getRuntime().availableProcessors() * 2;
        /**
         * BlockingQueue只是一个接口，有好多实现类
         * 1、ArrayBlockingQueue:底层基于数组实现的阻塞队列
         * 2、LinkedBlockingQueue：底层基于链表实现的阻塞队列
         * 3、LinkedBlockingDeque：底层基于双端链表实现的阻塞队列
         * 4、DelayQueue：延迟队列
         * 5、PriorityBlockingQueue:基于优先队列，底层是堆实现的。
         * 6、SynchronousQueue:同步队列,队列内只存储一个元素。公平模式。
         */
        BlockingQueue<Object> queue = new ArrayBlockingQueue<>(512);
        RejectedExecutionHandler policy = new ThreadPoolExecutor.DiscardPolicy();
//        executorService = new ThreadPoolExecutor(poolSize, poolSize,
//                0, TimeUnit.SECONDS,
//                queue,
//                policy);
        System.out.println(poolSize);
    }

    class ThreadPoolExecutor1 {
        /**
         * 线程池长期维持的线程数，即使没有任务，也不会回收
         */
        int corePoolSize;
        /**
         * 线程数的上限，比如一般是Integer.MAX_VALUE
         */
        int maximumPoolSize;
        /**
         * 超过corePoolSize的线程，会有一个活跃时间，超过这个时间，多余线程会被回收
         */
        long keepAliveTime;
        TimeUnit unit;
        /**
         * 任务的排队队列，也就是排期队列。
         */
        BlockingQueue<Runnable> workQueue;
        /**
         * 创建线程的渠道
         */
        ThreadFactory threadFactory;
        /**
         * 拒绝策略，也就是任务队列也塞满了，新来的任务需要被拒绝。
         */
        RejectedExecutionHandler handler;
    }

```

## 6、ThreadLocal

​		线程本地存储。ThreadLocal的作用是提供线程内的局部变量，<font color=red>这种变量在线程的生命周期内起作用，减少同一个线程内多个函数或者一个公共变量的传递复杂度</font>。

#### 6.1、ThreadLocalMap

- 每个线程中都有一个自己的ThreadLocalMap类对象，可以将线程自己的对象保持到其中，各管各的，线程可以正确的访问到自己的对象。

- 将一个公用的ThreadLocal静态实例作为key，将不同对象的引用保存到不同线程的ThreadLocalMap中，然后在线程执行的各处通过这个静态ThreadLocal实例的get()方法取的自己线程保存的对象，避免了将这个对象作为参数传递的麻烦。

- ThreadLocalMap就是线程里的一个属性，在Thread类定义

  ```java
  ThreadLocal.ThreadLocalMap threadLocals=null;
  ```

- 最常见的ThreadLocal使用场景，解决数据库连接，Session管理

  ```java
  private static final ThreadLocal local=new ThreadLocal();
      public static Session getSession(){
          Session s = (Session) local.get();
          try{
              if(s==null){
                  s= getSession()；
                  local.set(s);
              }
          }catch (HibernateException e){
  
          }
          return s;
      }
  ```

# 二、Dubbo的知识点

dubbo是一款高性能，轻量级的开源java RPC框架，提供3大核心功能：

- 面向接口的远程方法调用
- 智能容错和负载均衡
- 服务自动注册和发现

dubbo是一个分布式服务框架，用来提供高性能和透明化的RPC远程服务调用方案，以及SOA服务治理。

### 1、什么是RPC？

RPC：Remote Procedure Call，一个远程过程调用，它是一种通过网络从远程计算机程序上请求服务，而不需要了解底层网络技术的协议。比如说两个不同的服务A，B部署在两台不同的机器上，那么服务A如果想要调用服务B中的某个方法怎么解决？使用HTTP请求当然可以，但是可能会比较慢，而且一些优化做的并不好，RPC的出现就是为了解决这个问题。

RPC的原理：

- 服务消费方client调用以本地调用方式调用服务
- client stub接收到调用后负责将方法、参数等组装成能够进行网络传输的消息体。
- client stub找到服务地址，并将消息发送到服务端
- server stub收到消息后进行解码
- server stub根据解码结果调用本地的服务
- 本地服务执行并结果返回给server stub
- server stub将返回结果打包成消息并发送到消费方
- client stub接收到消息，并进行解码
- 服务消费方得到最终结果。

### 2、为什么要用dubbo？

dubbo和SOA分布式架构的流行有着关系。SOA面向服务的架构(Service Oriented Architecture)，就是把工程按照业务逻辑拆分成服务层，表现层两个工程。服务层中包含业务逻辑，只需要对外提供服务就行。表现层只需要处理和页面的交互，业务逻辑都是调用服务层的服务来实现。SOA架构中有两个主要角色：服务提供者Provieder和服务使用者Consumer

### 3、开发分布式及程序，可以直接基于HTTP接口进行通信，但是为什么要使用dubbo？

dubbo的四个特点：

- 负载均衡：同一个服务部署在不同的机器时该调用哪一台机器上的服务
- 服务调用链路生成：随着系统的发展，服务越来越多，服务间依赖关系变得错综复杂，甚至分不清哪个应用要在哪个应用之前启动，架构师都不能完整的描述应用的架构关系，dubbo可以为我们解决服务之间是如何互相调用的。
- 服务访问压力以及时长统计，资源调度和治理：基于访问压力实时管理集群容量，提供集群利用率。
- 服务降级：某个服务挂掉之后调用备用服务。

dubbo除了用在分布式系统中，也可以应用在微服务系统中，但是由于springboot cloud在微服务中更加广泛，所以一般提dubbo大部分在分布式系统的情况。

### 4、什么是分布式？

分布式或者说SOA分布式重要的就是面向服务，说简单的分布式就是我们把整个系统拆分成不同的服务然后将这些服务放在不同的服务器上减轻单体服务的压力提高并发量和性能。比如电商系统可以简单拆分成订单系统，商品系统，登录系统等，拆分之后的每个服务可以部署在不同的机器上，如果某一个服务的访问量比较大的话也可以将这个服务同时部署在多台机器上。

### 5、为什么要分布式？

从开发角度来讲单体应用的代码都集中在一起，而分布式系统的代码根据业务被拆分。所以每个团队可以负责一个服务的开发，这样提升了开发效率。另外，代码根据业务拆分之后更加便于维护和扩展。

另外，将系统拆分成分布式之后不光便于系统扩展和维护，更能提高整个系统的性能。把整个系统拆分成不同的服务、系统，然后每个服务，系统单独部署在一台服务器上，是不是很大成都提升了系统性能。

### 6、dubbo的架构

- Provider：暴露服务的服务提供方
- Consumer：调用远程服务的服务消费方
- Registry：服务注册与发现的注册中心
- Monitor：统计服务的调用次数和调用时间的监控中心
- Container：服务运行容器

#### 6.1、调用关系说明：

- Container服务容器负责启动，加载，运行服务提供者Provider。
- 服务提供者Provider在启动的是偶，向注册中心Registry注册自己提供的服务
- 服务消费者Consumer在启动的时候，向注册中心Registry订阅自己所需要的服务
- 注册中心Registry返回服务提供者Provider地址列表给消费者Consumer，如果有变更，注册中心Registry将基于长连接keep-alive推送变更数据给消费者Consumer
- 服务消费者Consumer，从服务提供者Provider地址列表中，基于软负载均衡算法，选一台提供者进行调用，如果调用失败，再选另一台调用。
  - dubbo提供了4中负载均衡算法：
    - 权重随机算法：RandomLoadBalance
    - 最少活跃调用数算法：LeastActiveLoadBalance
    - 一致性哈希算法：ConsistentHashLoadBalance
    - 加权轮询算法：RoundRobinLoadBalance

- 服务消费者Consumer和服务提供这Provider，在内存中累计调用次数和调用时间，定时每分钟发送一次统计数据到监控中心Monitor。

#### 6.2、总结：

- 注册中心负责服务地址的注册与查找，相当于目录服务，服务提供者Provider和服务消费者Consumer只在启动的时候与注册中心Registry交互，注册中心不转发请求，压力小。
- 监控中心Monitor负责统计各个服务调动次数，调用时间等，统计现在内存汇总后每分钟一次发送到监控中心服务器，并以报表的形式显示。
- 注册中心Registry，服务提供这Provider，服务消费者Consumer三者之间均为长连接Keep-alive，监控中心Monitor除外
- 注册中心Registry通过长连接感知服务提供者Provider的存在，服务提供者挂了，注册中心立即将推送事件通知消费者Consumer。
- 注册中心和监控中心全部挂了，不影响已经运行的服务提供者Provider和Consumer，消费者在本地缓存了提供者列表。
- 注册中心和监控中心都是可选的，服务消费者Consumer可以直接连服务提供者Provider
- 服务提供者Provider无状态，任意一台挂掉，不影响使用。
- 服务提供者Provider全部挂了，服务消费者Consumer应用将无法使用，并无限次重连等待服务提供者恢复。

#### 6.3、Zookeeper挂了与dubbo直连的情况

在实际生产中，如果zookeeper注册中心挂了，一段时间内服务消费者Consumer还是能够调用服务提供者Provider服务的，实际上Consumer使用了本地缓存进行通讯，Consumer本地缓存了提供者列表。

dubbo的健壮性:

- 监控中心挂了不影响使用