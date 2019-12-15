---
title: 10_Netty_Netty入门案例(源码分析)
date: 2019-12-14 17:38:17
tags: 
 - Netty
categories:
 - Netty
---

# 10_Netty_Netty入门案例（源码分析）

## Netty快速入门实例——TCP服务

1. Netty服务器在6688端口监听，客户端能发送消息给服务器“hello，服务器”；
2. 服务器可以回复消息给客户端“hello，客户端”；



## 服务端

步骤：

1. pom.xml添加netty-all依赖：

   ```xml
   <dependencies>
       <dependency>
           <groupId>io.netty</groupId>
           <artifactId>netty-all</artifactId>
           <version>4.1.43.Final</version>
       </dependency>
   </dependencies>
   ```

2. 编写主函数：

   ```java
   public class NettySever {
   
       public static void main(String[] args) throws InterruptedException {
           /*
               说明：
               1. 创建两个线程组BossGroup以及WorkGroup
               2. BossGroup只是处理连接请求，真正与客户端的业务处理，会交给WorkerGroup完成
               3. 两个都是无限循环
            */
           // 创建BossGroup 以及 WorkGroup
           EventLoopGroup bossGroup = new NioEventLoopGroup();
           EventLoopGroup workerGroup = new NioEventLoopGroup();
   
           try {
               // 创建服务器端的启动对象，配置参数
               ServerBootstrap bootstrap = new ServerBootstrap();
               // 使用链式编程的方式进行设置
               bootstrap.group(bossGroup,workerGroup)          // 设置两个线程组
                       .channel(NioServerSocketChannel.class)  // 使用NioServerSocketChannel作为服务器的通道实现
                       .option(ChannelOption.SO_BACKLOG,128)//设置线程队列等待连接个数
                       .childOption(ChannelOption.SO_KEEPALIVE,true)// 设置保持活动连接状态
                       .childHandler(new ChannelInitializer<SocketChannel>() { // 创建一个通道初始化对象（匿名对象）
                           // 向workerGroup关联的pipeline设置处理器
                           @Override
                           protected void initChannel(SocketChannel ch) throws Exception {
                               ch.pipeline().addLast(new NettyServerHandler());//添加处理器到pipeline尾部
                           }
                       });//给workerGroup的EventLoop对应的管道设置处理器
               System.out.println("==============服务器 is ready==============");
   
               // 绑定一个端口并同步，生成了一个ChannelFuture对象
               // 启动了服务器并绑定端口
               ChannelFuture cf = bootstrap.bind(6688).sync();
   
               // 对关闭通道进行监听
               cf.channel().closeFuture().sync();
           } catch (InterruptedException e) {
               e.printStackTrace();
           } finally {
               bossGroup.shutdownGracefully();
               workerGroup.shutdownGracefully();
           }
   
       }
   
   }
   ```

3. 编写自定义处理器

   ```java
   /**
    * 说明：
    * 1. 我们自定义一个Handler 需要继承netty规定好的某个HandlerAdapter适配器
    * 2. 这时我们自定义的Handler才能称之为一个Handler
    *
    * @author xieweidu
    * @createDate 2019-12-14 18:09
    */
   public class NettyServerHandler extends ChannelInboundHandlerAdapter {
   
       /**
        * 读取数据事件（这里我们可以读取客户端发送的数据）
        *
        * @param ctx ChannelHandlerContext：上下文对象，含有管道pipeline，通道channel，地址
        * @param msg Object：就是客户端发送的数据，默认是Object类型
        * @throws Exception
        */
       @Override
       public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
           System.out.println("读取事件发生=================");
           System.out.println("server ctx = " + ctx);
           // msg转为一个ByteBuffer
           // ByteBuf是netty提供的，不是NIO的ByteBuffer，Netty提供的性能更高
           ByteBuf buf = (ByteBuf) msg;
           System.out.println("客户端发送的消息是：" + buf.toString(CharsetUtil.UTF_8));
           System.out.println("客户端的地址是：" + ctx.channel().remoteAddress());
   
       }
   
       /**
        * 数据读取完毕，触发
        * @param ctx
        * @throws Exception
        */
       @Override
       public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
           // write+flush 将数据写入到缓存，并刷新
           // 一般来讲，我们对发送的数据进行编码
           ctx.writeAndFlush(Unpooled.copiedBuffer("hello,客户端",CharsetUtil.UTF_8));
       }
   
       /**
        * 处理异常，一般是需要关闭通道
        * @param ctx
        * @param cause
        * @throws Exception
        */
       @Override
       public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
           ctx.close();
       }
   }
   ```



## 客户端

1. 编写主函数

   ```java
   public class NettyClient {
   
       public static void main(String[] args) throws InterruptedException {
           // 客户端需要一个事件循环组
           EventLoopGroup eventExecutors = new NioEventLoopGroup();
   
           try {
               // 创建客户端启动对象
               // 注意客户端使用的是BootStrap，不是ServerBootStrap
               Bootstrap bootstrap = new Bootstrap();
   
               // 设置相关参数
               bootstrap.group(eventExecutors)// 设置线程组
                       .channel(NioSocketChannel.class)// 设置客户端实现类
                       .handler(new ChannelInitializer<SocketChannel>() {
                           @Override
                           protected void initChannel(SocketChannel ch) throws Exception {
                               ch.pipeline().addLast(new NettyClientHandler());// 加入自己的处理器
                           }
                       });
   
               System.out.println("客户端 is ok......");
               // 启动客户端去连接服务器端
               // 关于ChannelFuture，设计到hetty的异步模型
               ChannelFuture channelFuture = bootstrap.connect("127.0.0.1", 6688).sync();
               // 给关闭通道进行监听
               channelFuture.channel().closeFuture().sync();
           } finally {
               eventExecutors.shutdownGracefully();
           }
   
   
       }
   
   }
   ```

2. 编写自定义处理器

   ```java
   public class NettyClientHandler extends ChannelInboundHandlerAdapter {
       /**
        * 当通道就绪的时候就触发该方法
        *
        * @param ctx
        * @throws Exception
        */
       @Override
       public void channelActive(ChannelHandlerContext ctx) throws Exception {
           System.out.println("client :" + ctx);
           ctx.writeAndFlush(Unpooled.copiedBuffer("hello server！！！", CharsetUtil.UTF_8));
       }
   
       /**
        * 当通道有读取事件的时候，会触发该方法
        *
        * @param ctx
        * @param msg
        * @throws Exception
        */
       @Override
       public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
           ByteBuf buf = (ByteBuf) msg;
           System.out.println("服务器回复的消息：" + buf.toString(CharsetUtil.UTF_8));
           System.out.println("服务器的地址："+ctx.channel().remoteAddress());
       }
   
       @Override
       public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
           System.out.println("客户端异常发生");
           cause.printStackTrace();
           ctx.close();
       }
   }
   ```

   

## 源码分析

### 默认NioEventLoopGroup线程数

默认是CPU的核数*2，4核就是8个：

```java
DEFAULT_EVENT_LOOP_THREADS = Math.max(1, SystemPropertyUtil.getInt(        "io.netty.eventLoopThreads", NettyRuntime.availableProcessors() * 2));
```

## 服务器线程分配默认规则

比如计算机CPU有4核，默认开8个线程（也可以自行修改线程数），那么在服务器读取消息的时候打印线程信息，那么开多个客户端去连接，可以看到是轮询的规则；





