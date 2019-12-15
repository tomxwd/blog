---
title: 13_Netty_Http服务程序实例
date: 2019-12-15 11:47:20
tags: 
 - Netty
categories:
 - Netty
---

# 13_Netty_Http服务程序实例

要求：

1. Netty服务器在6688端口监听，浏览器发出请求http://localhost:6688
2. 服务器可以回复消息给客户端“hello！我是服务器”，并对特定请求资源进行过滤；

目的：

- Netty可以做Http服务开发，并且理解Handler实例以及客户端与发送请求的关系；



## 服务器主类

```java
public class TestHttpServer {

    public static void main(String[] args) {
        NioEventLoopGroup bossGroup = new NioEventLoopGroup(1);
        NioEventLoopGroup workerGroup = new NioEventLoopGroup();
        try {

            ServerBootstrap serverBootstrap = new ServerBootstrap();
            serverBootstrap.group(bossGroup, workerGroup)
                    .channel(NioServerSocketChannel.class)
                    .childHandler(new TestHttpServerInitializer());
            ChannelFuture channelFuture = serverBootstrap.bind(6688).sync();
            channelFuture.channel().closeFuture().sync();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            workerGroup.shutdownGracefully();
            bossGroup.shutdownGracefully();
        }
    }

}
```



## 初始化类

```java
public class TestHttpServerInitializer extends ChannelInitializer<SocketChannel> {

    @Override
    protected void initChannel(SocketChannel ch) throws Exception {
        // 向管道加入处理器

        // 得到管道
        ChannelPipeline pipeline = ch.pipeline();

        // 加入一个netty提供的HttpServerCodec codec => [coder  -  decoder]
        /**
         * HttpServerCodec说明：
         *  1. HttpServerCodec是netty提供的一个处理http的编码解码器
         */
        pipeline.addLast("MyHttpServerCode",new HttpServerCodec());
        //  2. 增加一个自定义的handler
        pipeline.addLast("MyTestHttpServerHandler",new TestHttpServerHandler());
    }
}
```



## 自定义处理器

```java
/**
 * 说明：
 * 1. SimpleChannelInboundHandler是ChannelInboundHandlerAdapter的子类
 * 2. HttpObject表示客户端和服务器端相互通讯的数据被封装成HttpObject类型
 *
 * @author xieweidu
 * @createDate 2019-12-15 11:51
 */
public class TestHttpServerHandler extends SimpleChannelInboundHandler<HttpObject> {

    /**
     * 当有读取事件的时候，就会触发
     *
     * @param ctx
     * @param msg
     * @throws Exception
     */
    @Override
    protected void channelRead0(ChannelHandlerContext ctx, HttpObject msg) throws Exception {
        // 判断msg是不是HttpRequest类型
        if (msg instanceof HttpRequest) {
            System.out.println("pipeline hashcode " + ctx.pipeline().hashCode() + " TestHttpServerHandler hashcode " + this.hashCode());
            System.out.println("msg 类型 = " + msg.getClass());
            System.out.println("客户端地址 " + ctx.channel().remoteAddress());

            // 获取到msg的信息
            HttpRequest httpRequest = (HttpRequest) msg;
            // 获取uri，过滤特定资源
            URI uri = new URI(httpRequest.uri());
            if ("/favicon.ico".equals(uri.getPath())) {
                System.out.println("你请求了favicon.ico,不做响应！");
                return;
            }


            // 回复信息给浏览器[Http协议]
            ByteBuf content = Unpooled.copiedBuffer("hello，我是服务器", CharsetUtil.UTF_8);
            // 构造一个Http的响应，即HttpResponse
            DefaultFullHttpResponse response = new DefaultFullHttpResponse(HttpVersion.HTTP_1_1, HttpResponseStatus.OK, content);
            response.headers()
                    .set(HttpHeaderNames.CONTENT_TYPE, "text/plain;charset=utf-8")
                    .set(HttpHeaderNames.CONTENT_LENGTH, content.readableBytes());
            // 将构建好的response返回
            ctx.writeAndFlush(response);
        }
    }
}
```













