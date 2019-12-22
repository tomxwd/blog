---
title: 18_Netty_Google_Protobuf编码器机制
date: 2019-12-18 22:26:02
tags: 
 - Netty
categories:
 - Netty
---

# 18_Netty_Google_Protobuf编码器机制

## 编码和解码的基本介绍

1. 编写网络应用程序的时候，因为数据在网络中传输的都是二进制字节码数据，在发送数据的时候就需要编码，接收数据的时候就需要解码；

2. codec（编解码器）的组成部分有两个：

   - encoder（编码器）

     encoder负责把**业务数据转换成字节码数据**；

   - decoder（解码器）

     decoder负责把**字节码数据转换成业务数据**；



## Netty本身的编码解码的机制与问题分析

1. Netty自身提供了一些codec（编解码器）
2. Netty提供的编码器：
   - StringEncoder，对字符串数据进行编码
   - ObjectEncoder，对Java对象进行编码
   - ......
3. Netty提供的解码器：
   - StringDecoder，对字符串数据进行解码
   - ObjectDecoder，对Java对象进行解码
   - ......
4. Netty本身自带的ObjectDecoder和ObjectEncoder可以用来实现pojo对象或各种业务对象的编码和解码，底层使用的仍然是Java序列化技术，而Java序列化技术本身效率就不高，存在如下问题：
   - **无法跨语言**
   - 序列化后的体积太大，是二进制编码的5倍多
   - 序列化性能太差

所以使用Google的Protobuf作为序列化技术



## Protobuf

### Protobuf基本介绍和使用示意图

1. Protobuf是Google发布的开源项目，全称Google Protocol Buffers，是一种轻便高效的结构化数据存储格式，可以用于结构化数据串行化，或者说序列化。它很适合做数据存储或**RPC数据交换格式**；

   目前很多公司http+json-->TCP+protobuf

2. [参考文档](https://developers.google.com/protocol-buffers/docs/proto)

3. Protobuf是以message的方式来管理数据的；

4. 支持跨平台、**跨语言**（支持目前绝大多数语言，例如C++、C#、Java、Python等）；

5. 高性能，高可靠性

6. 使用Protobuf编译器能自动生成代码，Protobuf是将类的定义使用.proto文件进行描述。【说明：在idea中编写.proto文件时，会自动提示是够下载Protobuf编写插件，可以让语法高亮；】

7. 然后通过protoc.exe编译器根据.proto自动生成.java文件

8. protobuf使用示意图；

   ![image-20191222181139733](18_Netty_Google_Protobuf%E7%BC%96%E7%A0%81%E5%99%A8%E6%9C%BA%E5%88%B6/image-20191222181139733.png)



### Protobuf快速入门实例1

编写程序，使用Protobuf完成如下功能：

1. 客户端可以发送一个Student Pojo 对象到服务器（通过Protobuf编码）
2. 服务器能接收Student Pojo对象，并显示信息（通过Protobuf编码）



**[ProtobufGithub下载地址](https://github.com/protocolbuffers/protobuf/releases/)**

需要下载两个，一个是java的，一个是win32的；



步骤：

1. 在pom.xml文件中引入Protobuf依赖
2. 编写proto文件Student.proto
3. 生成.java文件简单点的形式是，复制Student.proto到protoc-3.6.1-win32解压后的bin目录下，与protoc.exe同级，然后命令行中输入`protoc.exe --java_out=. Student.proto`生成StudentPOJO.java文件
   - 或者就复制到项目里运行生成；
4. 复制StudentPOJO.java到项目中；
5. 客户端发送对象到服务器，服务器接收
6. 客户端需要添加一个ProtobufEncode处理器，而服务端则添加ProtobufDecode处理器，并指定类型；



Student.proto文件：

```protobuf
syntax = "proto3"; // 版本
option java_outer_classname = "StudentPOJO"; // 生成的外部类名，同时也是文件名
// protobuf是以message管理数据
message Student {  // 会在外部类生成一个内部类Student，他是真正的发送POJO对象
    int32 id = 1;  // Student类中有一个属性名字为id，类型为int32（protobuf类型）1不代表值，表示属性的序号
    string name = 2;
}
```



客户端：

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
                            ch.pipeline()
                                    .addLast("encoder", new ProtobufEncoder())//在pipeline中加入ProtobufEncoder处理器
                                    .addLast(new NettyClientHandler());// 加入自己的处理器
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



客户端处理器：

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
        // 发送一个Student对象到服务器
        StudentPOJO.Student student = StudentPOJO.Student.newBuilder().setId(4).setName("豹子头林冲").build();
        ctx.writeAndFlush(student);
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



服务端：

```java
public class NettyServer {

    public static void main(String[] args) throws InterruptedException {
        /*
            说明：
            1. 创建两个线程组BossGroup以及WorkGroup
            2. BossGroup只是处理连接请求，真正与客户端的业务处理，会交给WorkerGroup完成
            3. 两个都是无限循环
            4. BossGroup和WorkerGroup含有的子线程（NioEventLoop）的个数，默认是CPU的核数*2
         */
        // 创建BossGroup 以及 WorkGroup
        EventLoopGroup bossGroup = new NioEventLoopGroup(1);
        EventLoopGroup workerGroup = new NioEventLoopGroup();

        try {
            // 创建服务器端的启动对象，配置参数
            ServerBootstrap bootstrap = new ServerBootstrap();
            // 使用链式编程的方式进行设置
            bootstrap.group(bossGroup, workerGroup)          // 设置两个线程组
                    .channel(NioServerSocketChannel.class)  // 使用NioServerSocketChannel作为服务器的通道实现
                    .option(ChannelOption.SO_BACKLOG, 128)//设置线程队列等待连接个数
                    .childOption(ChannelOption.SO_KEEPALIVE, true)// 设置保持活动连接状态
                    .childHandler(new ChannelInitializer<SocketChannel>() { // 创建一个通道初始化对象（匿名对象）
                        // 向workerGroup关联的pipeline设置处理器
                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {
                            System.out.println("客户端SocketChannel的hashCode：" + ch.hashCode());// 可以使用一个集合管理所有的Channel，在推送消息的时候，可以将业务加入到各个Channel对应的NioEventLoop的taskQueue或者scheduleTaskQueue
                            ch.pipeline()
                                    .addLast("decoder",new ProtobufDecoder(StudentPOJO.Student.getDefaultInstance()))//在pipeline中加入ProtobufDecoder处理器 这里需要指定对哪种对象进行解码
                                    .addLast(new NettyServerHandler());//添加处理器到pipeline尾部
                        }
                    });//给workerGroup的EventLoop对应的管道设置处理器
            System.out.println("==============服务器 is ready==============");

            // 绑定一个端口并同步，生成了一个ChannelFuture对象
            // 启动了服务器并绑定端口
            ChannelFuture cf = bootstrap.bind(6688).sync();

            // 给cf注册监听器，监控我们关心的事件
            cf.addListener(future -> {
                if(cf.isSuccess()){
                    System.out.println("监听端口6688成功！");
                }else{
                    System.out.println("监听端口6688失败！");
                }
            });

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



服务端处理器：

```java
public class NettyServerHandler extends SimpleChannelInboundHandler<StudentPOJO.Student> {

    @Override
    protected void channelRead0(ChannelHandlerContext ctx, StudentPOJO.Student student) throws Exception {
        System.out.println("客户端发送的数据 id=" + student.getId() + " 名字=" + student.getName());
    }

    /**
     * 读取数据事件（这里我们可以读取客户端发送的数据）
     *
     * @param ctx ChannelHandlerContext：上下文对象，含有管道pipeline，通道channel，地址
     * @param msg Object：就是客户端发送的数据，默认是Object类型
     * @throws Exception
     */
//    @Override
//    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
//        // 读取从客户端发送的StudentPOJO.Student
//        StudentPOJO.Student student = (StudentPOJO.Student) msg;
//        System.out.println("客户端发送的数据 id=" + student.getId() + " 名字=" + student.getName());
//    }

    /**
     * 数据读取完毕，触发
     *
     * @param ctx
     * @throws Exception
     */
    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
        // write+flush 将数据写入到缓存，并刷新
        // 一般来讲，我们对发送的数据进行编码
        ctx.writeAndFlush(Unpooled.copiedBuffer("hello,客户端", CharsetUtil.UTF_8));
    }

    /**
     * 处理异常，一般是需要关闭通道
     *
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



### Protobuf快速入门案例2

1. 客户端可以随机发送StudentPOJO/WorkerPOJO对象到服务器（通过Protobuf编码）
2. 服务端能接收StudentPOJO/WorkerPOJO对象（需要判断是哪种类型），并显示消息（通过Protobuf解码）



MyMessage.proto：

```protobuf
syntax = "proto3"; // 版本
option optimize_for = SPEED; // 快速解析
option java_package = "top.tomxwd.netty.codec2"; // 指定生成到哪个包下
option java_outer_classname = "MyDataInfo"; //外部类名称

// protobuf可以使用message管理其他的message
message MyMessage {
    // 定义一个枚举类型
    enum DataType {
        StudentType = 0; // 这里编号要从0开始
        WorkerType = 1;
    }

    // 用data_type来标识传的是哪一个枚举类型
    DataType data_type = 1;
    // 表示每次枚举类型最多只能出现其中的一个，节省空间
    oneof dataBody {
        Student student = 2;
        Worker worker = 3;
    }

}

message Student {  // 会在外部类生成一个内部类Student，他是真正的发送POJO对象
    int32 id = 1;  // Student类中有一个属性名字为id，类型为int32（protobuf类型）1不代表值，表示属性的序号
    string name = 2;
}
message Worker {
    string name = 1;
    int32 age = 2;
}
```



客户端：

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
                            ch.pipeline()
                                    .addLast("encoder", new ProtobufEncoder())//在pipeline中加入ProtobufEncoder处理器
                                    .addLast(new NettyClientHandler());// 加入自己的处理器
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



客户端处理器：

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
        // 随机发送Student或者Worker对象
        int random = new Random().nextInt(3);
        MyDataInfo.MyMessage myMessage = null;

        if (0 == random) {
            // 发送Student
            myMessage = MyDataInfo.MyMessage.newBuilder().setDataType(MyDataInfo.MyMessage.DataType.StudentType).setStudent(MyDataInfo.Student.newBuilder().setId(5).setName("玉麒麟 卢俊义").build()).build();
        } else {
            // 发送Worker
            myMessage = MyDataInfo.MyMessage.newBuilder().setDataType(MyDataInfo.MyMessage.DataType.WorkerType).setWorker(MyDataInfo.Worker.newBuilder().setAge(20).setName("老李").build()).build();
        }
        ctx.writeAndFlush(myMessage);
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
        System.out.println("服务器的地址：" + ctx.channel().remoteAddress());
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        System.out.println("客户端异常发生");
        cause.printStackTrace();
        ctx.close();
    }
}
```



服务器：

```java
public class NettyServer {

    public static void main(String[] args) throws InterruptedException {
        /*
            说明：
            1. 创建两个线程组BossGroup以及WorkGroup
            2. BossGroup只是处理连接请求，真正与客户端的业务处理，会交给WorkerGroup完成
            3. 两个都是无限循环
            4. BossGroup和WorkerGroup含有的子线程（NioEventLoop）的个数，默认是CPU的核数*2
         */
        // 创建BossGroup 以及 WorkGroup
        EventLoopGroup bossGroup = new NioEventLoopGroup(1);
        EventLoopGroup workerGroup = new NioEventLoopGroup();

        try {
            // 创建服务器端的启动对象，配置参数
            ServerBootstrap bootstrap = new ServerBootstrap();
            // 使用链式编程的方式进行设置
            bootstrap.group(bossGroup, workerGroup)          // 设置两个线程组
                    .channel(NioServerSocketChannel.class)  // 使用NioServerSocketChannel作为服务器的通道实现
                    .option(ChannelOption.SO_BACKLOG, 128)//设置线程队列等待连接个数
                    .childOption(ChannelOption.SO_KEEPALIVE, true)// 设置保持活动连接状态
                    .childHandler(new ChannelInitializer<SocketChannel>() { // 创建一个通道初始化对象（匿名对象）
                        // 向workerGroup关联的pipeline设置处理器
                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {
                            System.out.println("客户端SocketChannel的hashCode：" + ch.hashCode());// 可以使用一个集合管理所有的Channel，在推送消息的时候，可以将业务加入到各个Channel对应的NioEventLoop的taskQueue或者scheduleTaskQueue
                            ch.pipeline()
                                    .addLast("decoder", new ProtobufDecoder(MyDataInfo.MyMessage.getDefaultInstance()))//在pipeline中加入ProtobufDecoder处理器 这里需要指定对哪种对象进行解码
                                    .addLast(new NettyServerHandler());//添加处理器到pipeline尾部
                        }
                    });//给workerGroup的EventLoop对应的管道设置处理器
            System.out.println("==============服务器 is ready==============");

            // 绑定一个端口并同步，生成了一个ChannelFuture对象
            // 启动了服务器并绑定端口
            ChannelFuture cf = bootstrap.bind(6688).sync();

            // 给cf注册监听器，监控我们关心的事件
            cf.addListener(future -> {
                if (cf.isSuccess()) {
                    System.out.println("监听端口6688成功！");
                } else {
                    System.out.println("监听端口6688失败！");
                }
            });

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



服务器处理器：

```java
public class NettyServerHandler extends SimpleChannelInboundHandler<MyDataInfo.MyMessage> {

    @Override
    protected void channelRead0(ChannelHandlerContext ctx, MyDataInfo.MyMessage msg) throws Exception {
        // 根据dataType来显示不同的信息
        MyDataInfo.MyMessage.DataType dataType = msg.getDataType();
        if (dataType == MyDataInfo.MyMessage.DataType.StudentType) {
            MyDataInfo.Student student = msg.getStudent();
            System.out.println("学生id:" + student.getId() + " 名字:" + student.getName());
        } else if (dataType == MyDataInfo.MyMessage.DataType.WorkerType) {
            MyDataInfo.Worker worker = msg.getWorker();
            System.out.println("工人年龄:" + worker.getAge() + " 名字:" + worker.getName());
        } else {
            System.out.println("传输的类型不正确");
        }
    }

    /**
     * 读取数据事件（这里我们可以读取客户端发送的数据）
     *
     * @param ctx ChannelHandlerContext：上下文对象，含有管道pipeline，通道channel，地址
     * @param msg Object：就是客户端发送的数据，默认是Object类型
     * @throws Exception
     */
//    @Override
//    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
//        // 读取从客户端发送的StudentPOJO.Student
//        StudentPOJO.Student student = (StudentPOJO.Student) msg;
//        System.out.println("客户端发送的数据 id=" + student.getId() + " 名字=" + student.getName());
//    }

    /**
     * 数据读取完毕，触发
     *
     * @param ctx
     * @throws Exception
     */
    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
        // write+flush 将数据写入到缓存，并刷新
        // 一般来讲，我们对发送的数据进行编码
        ctx.writeAndFlush(Unpooled.copiedBuffer("hello,客户端", CharsetUtil.UTF_8));
    }

    /**
     * 处理异常，一般是需要关闭通道
     *
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







