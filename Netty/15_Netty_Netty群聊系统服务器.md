---
title: 15_Netty_Netty群聊系统服务器
date: 2019-12-15 14:07:05
tags: 
 - Netty
categories:
 - Netty
---

# 15_Netty_Netty群聊系统服务器

实例要求：

1. 编写一个Netty群聊系统，实现服务器端和客户端之间的数据简单通讯（非阻塞）
2. 实现多人群聊
3. **服务器端：**可以监测用户上线、离线，并实现消息转发功能；
4. **客户端：**通过Channel可以无阻塞发送消息给其他所有用户，同时可以接受其他用户发送的消息（由服务器转发得到）

## 服务器端

服务器端主服务类：

```java
/**
 * 群聊系统服务端
 *
 * @author xieweidu
 * @createDate 2019-12-16 22:32
 */
public class GroupChatServer {

    /**
     * 监听端口
     */
    private int port;

    public GroupChatServer(int port) {
        this.port = port;
    }

    /**
     * 编写run方法，处理客户端的请求
     */
    public void run() throws InterruptedException {
        // 创建两个线程组
        NioEventLoopGroup bossGroup = new NioEventLoopGroup(1);
        NioEventLoopGroup workerGroup = new NioEventLoopGroup(8);
        try {
            // 启动类
            ServerBootstrap serverBootstrap = new ServerBootstrap();
            // 设置参数
            serverBootstrap
                    .group(bossGroup, workerGroup)
                    .channel(NioServerSocketChannel.class)
                    .option(ChannelOption.SO_BACKLOG, 128)
                    .childOption(ChannelOption.SO_KEEPALIVE, true)
                    .childHandler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {
                            // 获取Pipeline
                            // 向Pipeline加入一个解码器以及一个编码器,以及自己的业务处理Handler
                            ch.pipeline()
                                    .addLast("decoder", new StringDecoder())
                                    .addLast("encoder", new StringEncoder())
                                    .addLast(new GroupChatServerHandler());
                        }
                    });
            System.out.println("netty群聊服务器启动");
            ChannelFuture channelFuture = serverBootstrap.bind(port).sync();
            // 监听关闭事件
            channelFuture.channel().closeFuture().sync();
        } finally {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }

    public static void main(String[] args) throws InterruptedException {
        new GroupChatServer(7000).run();
    }

}
```

服务器端自定义处理器：

```java
public class GroupChatServerHandler extends SimpleChannelInboundHandler<String> {

    /**
     * 定义一个Channel组，管理所有的Channel
     * GlobalEventExecutor.INSTANCE是全局的事件执行器，是一个单例
     */
    private static ChannelGroup channelGroup = new DefaultChannelGroup(GlobalEventExecutor.INSTANCE);
    private SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");

    /**
     * handlerAdded表示连接建立，一旦连接，第一个被执行的方法
     * 在这里把Channel加入到ChannelGroup
     *
     * @param ctx
     * @throws Exception
     */
    @Override
    public void handlerAdded(ChannelHandlerContext ctx) throws Exception {
        Channel channel = ctx.channel();
        channelGroup.add(channel);
        // 将该客户端的信息推送给其他在线的客户端,channelgroup的writeAndFlush会遍历所有channel并发送消息
        channelGroup.writeAndFlush(sdf.format(new Date()) + "【客户端】" + channel.remoteAddress() + "加入聊天\n");
    }

    /**
     * 断开连接，将XXX客户离开信息推送给当前在线的客户
     * handlerRemoved每执行一次，channelGroup中的channel也会被移除，不需要手动调用remove方法
     *
     * @param ctx
     * @throws Exception
     */
    @Override
    public void handlerRemoved(ChannelHandlerContext ctx) throws Exception {
        Channel channel = ctx.channel();
        channelGroup.writeAndFlush(sdf.format(new Date()) + "【客户端】" + channel.remoteAddress() + "离开了\n");
        System.out.println("当前ChannelGroup Size：" + channelGroup.size());
    }

    /**
     * channelActive表示channel处于活动状态，提示XXX上线了
     *
     * @param ctx
     * @throws Exception
     */
    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        System.out.println(ctx.channel().remoteAddress() + " 上线了");
    }

    /**
     * channelInactive表示channel处于不活动状态，提示XXX下线了
     *
     * @param ctx
     * @throws Exception
     */
    @Override
    public void channelInactive(ChannelHandlerContext ctx) throws Exception {
        System.out.println(ctx.channel().remoteAddress() + " 离线了");
    }

    /**
     * 读取数据并转发给在线客户
     *
     * @param ctx
     * @param msg
     * @throws Exception
     */
    @Override
    protected void channelRead0(ChannelHandlerContext ctx, String msg) throws Exception {
        // 获取当前channel
        Channel channel = ctx.channel();
        // 遍历channelGroup，根据不同的情况，回送不同的消息
        channelGroup.forEach(ch -> {
            if (channel != ch) {
                // 非当前channel
                ch.writeAndFlush(sdf.format(new Date()) + "【客户】" + channel.remoteAddress() + "发送消息：" + msg + "\n");
            } else {
                // 自己发送的消息
                ch.writeAndFlush(sdf.format(new Date()) + "【我】发送消息：" + msg + "\n");
            }
        });
    }

    /**
     * 发生异常事件
     *
     * @param ctx
     * @param cause
     * @throws Exception
     */
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        // 关闭通道
        ctx.close();
    }
}
```



## 客户端

客户端主类：

```java
/**
 * netty群聊 客户端
 *
 * @author xieweidu
 * @createDate 2019-12-16 23:39
 */
public class GroupChatClient {

    /**
     * 主机信息
     */
    private final String host;
    /**
     * 端口信息
     */
    private final int port;


    public GroupChatClient(String host, int port) {
        this.host = host;
        this.port = port;
    }

    public void run() throws InterruptedException {
        NioEventLoopGroup group = new NioEventLoopGroup(8);
        try {
            Bootstrap bootstrap = new Bootstrap()
                    .group(group)
                    .channel(NioSocketChannel.class)
                    .handler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {
                            // 得到pipeline，加入相关的handler
                            ch.pipeline()
                                    .addLast(new StringDecoder())
                                    .addLast(new StringEncoder())
                                    .addLast(new GroupChatClientHandler());
                        }
                    });
            ChannelFuture channelFuture = bootstrap.connect(host, port).sync();
            // 得到Channel
            Channel channel = channelFuture.channel();
            System.out.println("--------------" + channel.localAddress() + "--------------");
            // 客户端需要输入信息，创建一个扫描器
            Scanner scanner = new Scanner(System.in);
            while (scanner.hasNext()) {
                String msg = scanner.nextLine();
                // 通过channel 发送到服务器端
                channel.writeAndFlush(msg + "\r\n");
            }
        } catch (InterruptedException e) {
            group.shutdownGracefully();
        }
    }

    public static void main(String[] args) throws InterruptedException {
        new GroupChatClient("127.0.0.1", 7000).run();
    }

}
```

客户端自定义处理器：

```java
/**
 * 服务端Handler
 *
 * @author xieweidu
 * @createDate 2019-12-16 23:49
 */
public class GroupChatClientHandler extends SimpleChannelInboundHandler<String> {

    @Override
    protected void channelRead0(ChannelHandlerContext ctx, String msg) throws Exception {
        System.out.println(msg.trim());
    }
}
```



## 扩展（私聊功能思路）

1. 首先使用HashMap管理，key为用户id，value为用户的channel；
2. 私聊的时候先指定用户id，然后再发送消息；
3. 服务器接收到消息，再将消息转发给对应用户的对应channel即可实现私聊功能；

