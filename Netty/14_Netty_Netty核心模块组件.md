---
title: 14_Netty_Netty核心模块组件
date: 2019-12-15 14:07:05
tags: 
 - Netty
categories:
 - Netty
---

# 14_Netty_Netty核心模块组件

## Bootstrap、ServerBootstrap

1. Bootstrap意思是引导，一个Netty应用通常由一个Bootstrap开始，主要作用是配置整个Netty程序，串联各个组件，Netty中Bootstrap类是客户端程序的启动引导类，ServerBootstrap是服务端启动引导类；
2. 常见的方法：
   - public ServerBootstrap **group**(EventLoopGroup parentGroup, EventLoopGroup childGroup)，该方法用于服务器端，用来设置两个EventLoop；
   - public B **group**(EventLoopGroup group)，该方法用于客户端，用来设置一个EventLoopGroup；
   - public B **channel**(Class<? extends C> channelClass)，该方法用来设置一个服务器端的通道实现；
   - public <T\> B **option**(ChannelOption<T\> option, T value)，用来给ServerChannel添加配置；
   - public <T\> ServerBootstrap **childOption**(ChannelOption \<T> childOption,T value)，用来给接收到的通道添加配置；
   - public ServerBootstrap **childHandler**(ChannelHandler childHandler)，该方法用来设置业务处理类（自定义handler）；
     - 这里注意，如果是handler，则是给parentGroup，也就是BossGroup添加处理类
   - public ChannelFuture bind(int inetPort)，该方法用于服务器端，用来设置占用的端口号；
   - public ChannelFuture connect(String inetHost, int InetPort)，该方法用于客户端，用来连接服务器端；



## Future、ChannelFuture

1. Netty中所有的IO操作都是异步的，不能立刻得知消息是否被正确处理，但是可以过一会等它执行完成或直接注册一个监听，具体的实现就是通过Future和ChannelFuture，他们可以注册一个监听，当操作执行成功或失败的时候监听会自动触发注册的监听事件；
2. 常见的方法：
   - Channel channel()，返回当前正在进行IO操作的通道；
   - ChannelFuture sync()，等待异步操作执行完毕；



## Channel

1. Netty网络通信的组件，能够用于执行网络I/O操作；

2. 通过Channel可以获得当前网络连接的通道的状态；

3. 通过Channel可以获得当前网络连接的配置参数（例如接收缓冲区的大小）；

4. Channel提供异步的网络I/O操作（如建立连接、读写、绑定端口），异步调用意味着任何I/O调用都将立即返回，并且不保证在调用结束时所请求的I/O操作已完成；

5. 调用立即返回一个ChannelFuture实例，通过注册监听器到ChannelFuture上，可以I/O操作成功、失败或取消时回调通知调用方；

6. 支持关联I/O操作与对应的处理程序；

7. 不同协议、不同的阻塞类型的连接都有不同的Channel类型与之对应，常用的Channel类型：

   - **NioSocketChannel：**异步的客户端TCP Socket连接；
   - **NioServerSocketChannel：**异步的服务器端TCP Socket连接；
   - **NioDatagramChannel：**异步的UDP连接；
   - **NioSctpChannel：**异步的客户端Sctp连接；
   - **NioSctpServerChannel：**异步的服务器端Sctp连接；

   这些通道涵盖了UDP和TCP网络IO以及文件IO；



## Selector

1. Netty基于Selector对象实现I/O多路复用，通过Selector一个线程可以监听多个连接的Channel事件；
2. 当向一个Selector中注册Channel后，Selector内部的机制就可以自动不断地查询（Select）这些注册的Channel是否有已就绪的I/O事件（例如可读、可写、网络连接完成等），这样程序就可以很简单的使用一个线程高效地管理多个Channel；



## ChannelHandler及其实现类

1. ChannelHandler是一个接口，处理I/O事件或拦截I/O操作，并将其转发到其ChannelPipeline（业务处理链）中的下一个处理程序；

2. ChannelHandler本身并没有提供很多方法，因为这个接口有许多的方法需要实现，使用期间，可以继承它的子类；

3. ChannelHandler机器实现类一览图：

   ![image-20191215153416101](14_Netty_Netty%E6%A0%B8%E5%BF%83%E6%A8%A1%E5%9D%97%E7%BB%84%E4%BB%B6/image-20191215153416101.png)

   - ChannelInboundHandler用于处理入站I/O事件；
   - ChannelOutboundHandler用于处理出站I/O操作；

   适配器：

   - ChannelInboundHandlerAdapter用于处理入站I/O事件；
   - ChanneloutboundHandlerAdapter用于处理出站I/O操作；
   - ChannelDuplexHandler用于处理入站和出站事件；

4. 我们经常需要自定义一个Handler类去继承ChannelInboundHandlerAdapter，然后通过重写相应方法实现业务逻辑，一般重写以下方法：

   - channelRegistered：通道注册事件

   - channelActive：通道就绪事件
   - channelRead：通道读取数据事件
   - channelReadComplete：数据读取完毕事件
   - exceptionCaught：通道发生异常事件
   - handlerAdded：加入了新的处理器
   - handlerRemoved：移除了处理器



## Pipeline和ChannelPipeline

ChannelPipeline是一个重点：

1. ChannelPipeline是一个Handler的集合，它负责处理和拦截inbound或者outbound的事件和操作，相当于一个贯穿Netty的链。**（也可以这样理解：ChannelPipeline是保存ChannelHandler的List，用于处理或者拦截Channel的入站事件和出站操作）**

2. ChannelPipeline实现了一种高级形式的拦截过滤器模式，使用户可以完全控制事件的处理方式，以及Channel中各个的ChannelHandler如何相互交互；

3. 在Netty中每个Channel都有且仅有一个ChannelPipeline与之对应，它们的组成关系如下：

   ![image-20191215155618328](14_Netty_Netty%E6%A0%B8%E5%BF%83%E6%A8%A1%E5%9D%97%E7%BB%84%E4%BB%B6/image-20191215155618328.png)

   - 一个Channel包含了一个ChannelPipeline，而ChannelPipeline中又维护了一个由ChannelHandlerContext组成的双向链表，并且每个ChannelHandlerContext中又关联着一个ChannelHandler。
   - 入站事件和出站事件在一个双向链表中，入站事件会从链表head往后传递到最后一个入站的handler。出站事件会从链表tail往前传递到最前一个出站的handler，两种类型的handler互不干扰；
     - 这也就说明了为什么Server到Client是从head到tail，而Client是tail到head，Server到Client是入站，Client到Server是出站；

4. 常用方法：

   - ChannelPipeline addFirst(ChannelHandler...handlers)：把一个业务处理类（handler）添加到链表中的第一个位置；
   - ChannelPipeline addLast(ChannelHandler...handlers)：把一个业务处理类（handler）添加到链表中的最后一个位置；



## ChannelHandlerContext

1. 保存Channel相关的所有上下文信息，同时关联一个ChannelHandler对象；
2. 即ChannelHandlerContext中包含一个具体的事件处理器ChannelHandler，同时ChannelHandlerContext中也绑定了对应的pipeline以及Channel信息，方便对ChannelHandler进行调用；
3. 常用方法：
   - ChannelFuture close()：关闭通道
   - ChannelOutboundInvoker flush()：刷新
   - ChannelFuture writeAndFlush(Object msg)：将数据写入到ChannelPipeline中，当前ChannelHandler的下一个ChannelHandler开始处理（出站）



## ChannelOption

1. Netty在创建Channel实例后，一般都需要设置ChannelOption参数

2. ChannelOption参数如下：

   - **ChannelOption.So_BACKLOG**

     对应TCP/IP协议listen函数中的backlog参数，用来初始化服务器可连接队列大小。

     服务端处理客户端请求是顺序处理的，所以同一时间只能处理一个客户端连接。

     多个客户端来的时候，服务端将不能处理的客户端连接请求放在队列中等待处理，backlog参数指定了队列的大小；

   - **ChannelOption.SO_KEEPALIVE**

     一直保持连接活动状态；



## EventLoopGroup和其实现类NioEventLoopGroup

1. EventLoopGroup是一组EventLoop的抽象，Netty为了更好的利用多核CPU资源，一般会有多个EventLoop同时工作，每个EventLoop维护着一个Selector实例；

2. EventLoopGroup提供next接口，可以从组里按照一定规则获取其中一个EventLoop来处理任务。【在Netty服务器编程中，我们一般需要提供两个EventLoopGroup，例如：BossEventLoopGroup和WorkEventLoopGroup。

3. 通常一个服务端口即一个ServerSocketChannel对应一个Selector和一个EventLoop线程。BossEventLoop负责接收客户端的连接并将SocketChannel交给WorkerEventLoopGroup来进行IO处理；

   ![image-20191215190543392](14_Netty_Netty%E6%A0%B8%E5%BF%83%E6%A8%A1%E5%9D%97%E7%BB%84%E4%BB%B6/image-20191215190543392.png)

   - BossEventLoopGroup通常是一个单线程的EventLoop，EventLoop维护着一个注册了ServerSocketChannel的Selector实例，BossEventLoop不断轮询Selector，将连接事件分离出来；通常是OP_ACCEPT事件，然后将接收到的SocketChannel交给WorkerEventLoopGroup；
   - WorkerEventLoopGroup会由next选择其中一个EventLoop来将这个SocketChannel注册到其维护的Selector并对其后续的IO事件进行处理；

4. 常用方法：

   - public NioEventLoopGroup()：构造方法
   - public Future<?> shutdownGracefully()：断开连接，关闭线程
   - 