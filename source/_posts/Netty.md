---
title: Netty
date: 2018-02-15 10:10:28
tags: Java
---

# 一、Netty

### 1、Netty原理

​		Netty是一个高性能、异步事件驱动的NIO框架，基于Java NIO提供的API实现。它提供了TCP、UDP和文件传输的支持，作为一个异步NIO框架，Netty的所有IO操作都是异步非阻塞的，通过Future-Listener机制，用户可以方便的主动获取或者通过通知机制获得IO操作结果。

### 2、Netty高性能

​		在IO编程中，当需要同时处理多个客户端接入请求时，可以用多线程或者IO多路服用技术进行处理。IO多路服用技术通过把多个IO的阻塞复用到同一个select的阻塞上，从而使得系统在单线程的情况下可以同时处理多个客户端请求。与传统的多线程、多进程模型相比，IO多路复用的最大优势是系统开销小，系统不需要创建新的额外进程或者线程，也不需要维护这些进程和线程的运行，降低了系统的维护工作量，节省了系统资源。

​		与Socket类和ServerSocket类相对应，NIO也提供了SocketChannel和ServerSocketChannel两种不同的套接字通道实现。

#### 2.1、多路复用通讯方式

​		Netty架构按照Reactor模式设计和实现。

##### 1、服务端通信

- NIO Server打开ServerSocketChannel
- 绑定监听地址InetSocketAddress
- Reactor Thread创建Selector，启动线程
- NIO Server将ServerSocketChannel注册到Selector，监听SelectionKey.OP_ACCEPT
- Selector轮询就绪的Key
- handleAccept()处理新的客户端介入
- IOHandler设置新建客户端连接的Socket
- 向Selector注册监听读操作SelectionKey.OP_READ
- handleRead()异步读请求消息到ByteBuffer
- IOHandler开始decode请求
- 然后异步写ByteBuffer到SocketChannel

##### 2、客户端通信

- NIO  Client打开SocketChannel
- 设置SocketChannel为非阻塞模式，同时设置TCP参数
- 异步连接服务端Server
- 判断连接结果，如果连接成功，则向多路复用器注册读事件OP_READ，如果连接失败，则向Reactor线程的多路复用器注册OP_CONNECT事件
- 然后Reactor Thread创建Selector，启动线程
- Selector轮询就绪的Key
- handlerConnect()连接IOHandler
- 判断连接是否完成，如果完成则执行向多路复用器注册读事件OP_READ
- handlerRead()异步读请求消息到ByteBuffer
- 然后在decode请求消息
- 异步写ByteBuffer到SocketChannel

Netty的IO线程NioEventLoop由于聚合了多路复用器Selector，可以同时并发处理成百上千个客户端Channel，由于读写操作都是非阻塞的，这就可以充分提升IO线程的运行效率。避免由于频繁IO阻塞导致的线程挂起。

#### 2.2、异步通讯NIO

​		由于Netty采用异步