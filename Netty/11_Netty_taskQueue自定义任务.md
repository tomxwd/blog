---
title: 11_Netty_taskQueue自定义任务
date: 2019-12-15 09:16:17
tags: 
 - Netty
categories:
 - Netty
---

# 11_Netty_taskQueue自定义任务

任务队列中的Task有3种典型使用场景

1. **用户程序自定义的普通任务**

   这里有一个非常耗费时间的业务   ->  异步执行    -> 提交到该Channel对应的NioEventLoop的**taskQueue**中即可；

   服务器端读取到数据的时候处理方法：

   ```java
   @Override
   public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
       // 这里有一个非常耗费时间的业务   ->  异步执行    -> 提交到该Channel对应的NioEventLoop的taskQueue中即可
   
       // 解决方案1：用户程序自定义的普通任务
       ctx.channel().eventLoop().execute(()->{
           try {
               Thread.sleep(10*1000);
               ctx.writeAndFlush(Unpooled.copiedBuffer("hello,服务端10秒了",CharsetUtil.UTF_8));
           } catch (InterruptedException e) {
               e.printStackTrace();
           }
       });
       System.out.println("go on...");
   }
   ```

   如果是多个普通任务：

   ```java
   ctx.channel().eventLoop().execute(()->{
       try {
           Thread.sleep(10*1000);
           ctx.writeAndFlush(Unpooled.copiedBuffer("hello,服务端10秒了",CharsetUtil.UTF_8));
       } catch (InterruptedException e) {
           e.printStackTrace();
       }
   });
   ctx.channel().eventLoop().execute(()->{
       try {
           Thread.sleep(20*1000);
           ctx.writeAndFlush(Unpooled.copiedBuffer("hello,服务端30秒了",CharsetUtil.UTF_8));
       } catch (InterruptedException e) {
           e.printStackTrace();
       }
   });
   ```

   将会按顺序执行，比如第一个10秒，第二个20秒，那么第二个真正执行完的时间是30秒，因为他们是在同一个线程里的；

2. **用户自定义定时任务**

   用户自定义定时任务    ->  该任务是提交到**scheduleTaskQueue**中；

   ```java
   // 用户自定义定时任务    ->  该任务是提交到scheduleTaskQueue中
   ctx.channel().eventLoop().schedule(()->{
       ctx.writeAndFlush(Unpooled.copiedBuffer("hello,服务端5秒了",CharsetUtil.UTF_8));
   },5, TimeUnit.SECONDS);
   ```

   跟第一种类似，不过是丢到了scheduleTaskQueue中；	

3. **非当前Reactor线程调用Channel的各种方法**

   可以在初始化Channel的时候，存起客户端的SocketChannel，在推送的时候将业务塞给各个Channel的queue中即可；

   ```java
   @Override
   protected void initChannel(SocketChannel ch) throws Exception {
       System.out.println("客户端SocketChannel的hashCode：" + ch.hashCode());// 可以使用一个集合管理所有的Channel，在推送消息的时候，可以将业务加入到各个Channel对应的NioEventLoop的taskQueue或者scheduleTaskQueue
       ch.pipeline().addLast(new NettyServerHandler());//添加处理器到pipeline尾部
   }
   ```



## Netty模型再说明

1. Netty抽象出两组**线程池**，BossGroup专门负责接收客户端连接，**WorkerGroup**专门负责网络读写操作；
2. NioEventLoop表示一个不断循环执行处理任务的线程，每个NioEventLoop都有一个Selector，用于监听绑定在其上的Socket网络通道；
3. NioEventLoop内部采用**串行化**设计，从消息的读取->解码->处理->编码->发送，始终由IO线程NioEventLoop负责；



- NioEventLoopGroup下包含多个NioEventLoop

- 每个NioEventLoop中包含有一个Selector，一个taskQueue

- 每个NioEventLoop的Selector上可以注册监听多个NioChannel

- 每个NioChannel只会绑定在唯一一个NioEventLoop上

- 每个NioChannel都绑定有一个自己的ChannelPipeline

  