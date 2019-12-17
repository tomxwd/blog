---
title: 09_Netty_Netty模型
date: 2019-12-14 12:05:18
tags: 
 - Netty
categories:
 - Netty
---

# 09_Netty_Netty模型

## 通俗版

Netty主要基于主从Reactors多线程模型做出了一定的改进，其中主从Reactor多线程模型有多个Reactor；

![Netty模型通俗版](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/Netty/09Netty%E6%A8%A1%E5%9E%8B%E9%80%9A%E4%BF%97%E7%89%88.png)

1. BossGroup线程维护Selector，只关注accept事件；
2. 接收到accept事件后，获取到对应的SocketChannel，进一步封装成NIOSocketChannel，并注册到WorkerGroup线程（事件循环），并进行维护；
3. 当Worker线程监听到Selector中的通道发生了自己感兴趣的事件后，就进行处理（就由Handler完成），这里Handler已经加入到通道中了；



## 进阶版

![Netty模型进阶版](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/Netty/09Netty%E6%A8%A1%E5%9E%8B%E8%BF%9B%E9%98%B6%E7%89%88.png)



## 详细版

 ![Netty模型详细版](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/Netty/09Netty%E6%A8%A1%E5%9E%8B%E8%AF%A6%E7%BB%86%E7%89%88.jfif)

1. Netty抽象出两组线程池，BossGroup专门负责客户端的连接，WorkGroup专门负责网络的读写；
2. BossGroup和WorkGroup类型都是NioEventLoopGroup；
3. NioEventLoopGroup相当于一个事件循环组，这个组中含有多个事件循环，每一个事件循环是NioEventLoop；
4. NioEventLoop表示一个不断循环的执行处理任务的线程，每个NioEventLoop都有一个Selector，用于监听绑定在其上的socket的网络通讯；
5. NioEventLoopGroup可以有多个线程，即可以含有多个NioEventLoop；
6. 每个Boss下的NioEventGroup循环执行的步骤有三步：
   - 轮询accept事件
   - 处理accept事件，与client建立连接，生成NioSocketChannel，并将其注册到某个worker NioEventLoop上的selector
   - 处理任务队列的任务，即runAllTasks；
7. 每个Worker NioEventLoop循环执行的步骤
   - 轮询read/write事件
   - 处理i/o事件，即read/write事件，在对应的NioSocketChannel上进行处理；
   - 处理任务队列的其他任务，即runAllTasks；
8. 每一个Wroker NioEventLoop处理业务的时候，会使用pipeline（管道），pipeline中包含了Channel，即通过pipeline可以获取对应的通道，pipeline中维护了很多处理器Handler，这些处理器可以对数据进行一系列的处理；

