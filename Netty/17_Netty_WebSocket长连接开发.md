---
title: 17_Netty_WebSocket长连接开发
date: 2019-12-18 20:36:20
tags: 
 - Netty
categories:
 - Netty
---

# 17_Netty_WebSocket长连接开发

**实例要求：**

1. Http协议是无状态的，浏览器和服务器间的请求响应一次，下一次会重新创建连接；
2. 实现基于WebSocket的长连接的全双工的交互；
3. 改变Http协议多次请求的约束，实现长连接，服务器可以发送消息给浏览器；
4. 客户端浏览器和服务器端会相互感知，比如服务器关闭了，浏览器会感知，同样浏览器关闭了，服务器也会感知；



## 服务器端

主启动类：

```java
public class MyWebSocketServer {

    public static void main(String[] args) {
        NioEventLoopGroup bossGroup = new NioEventLoopGroup(1);
        NioEventLoopGroup workerGroup = new NioEventLoopGroup(8);
        try {
            ServerBootstrap serverBootstrap = new ServerBootstrap()
                    .group(bossGroup,workerGroup)
                    .channel(NioServerSocketChannel.class)
                    .handler(new LoggingHandler(LogLevel.INFO))
                    .childHandler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {
                            ChannelPipeline pipeline = ch.pipeline();
                            /**
                             *  1. 因为基于Http协议，所以要使用Http的编解码器
                             *  2. 而且是以块方式写的，所以添加ChunkedWrite处理器
                             *  3. 因为Http数据在传输过程中是分段的，所以用HttpObjectAggregator，
                             *     把多个段聚合，这就是为什么当浏览器发送大量数据的时候，就会发出多次Http请求
                             *  4. 对于WebSocket，它的数据是以帧（frame）的形式传递的，可以看到
                             *     WebSocketFrame下面有六个子类
                             *  5. 浏览器请求的时候ws://localhost:7000/XXX（表示请求的uri）
                             *  6. WebSocketServerProtocolHandler核心功能是将Http协议升级为ws协议，保持长连接
                             *  7. 最后添加一个自定义handler，用于处理具体业务逻辑
                             *  8. 是通过一个状态码101
                             */
                            pipeline.addLast(new HttpServerCodec())
                                    .addLast(new ChunkedWriteHandler())
                                    .addLast(new HttpObjectAggregator(8192))
                                    .addLast(new WebSocketServerProtocolHandler("/hello"))
                                    .addLast(new MyTextWebSocketFrameHandler());
                        }
                    });
            ChannelFuture channelFuture = serverBootstrap.bind(7000).sync();
            channelFuture.channel().closeFuture().sync();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }

}
```



自定义处理器：

```java
/**
 * 这里用的泛型TextWebSocketFrame类型，表示一个文本帧（frame）
 *
 * @author xieweidu
 * @createDate 2019-12-18 21:39
 */
public class MyTextWebSocketFrameHandler extends SimpleChannelInboundHandler<TextWebSocketFrame> {

    /**
     * Web客户端连接后，第一个会触发这个方法
     *
     * @param ctx
     * @throws Exception
     */
    @Override
    public void handlerAdded(ChannelHandlerContext ctx) throws Exception {
        // id表示唯一的值，LongText是唯一的,ShortText有可能重复
        System.out.println("MyTextWebSocketFrameHandler.handlerAdded被调用 " + ctx.channel().id().asLongText());
        System.out.println("MyTextWebSocketFrameHandler.handlerAdded被调用 " + ctx.channel().id().asShortText());
    }

    @Override
    public void handlerRemoved(ChannelHandlerContext ctx) throws Exception {
        System.out.println("MyTextWebSocketFrameHandler.handlerRemoved被调用" + ctx.channel().id().asLongText());
        System.out.println("MyTextWebSocketFrameHandler.handlerRemoved被调用" + ctx.channel().id().asShortText());
    }

    /**
     * 发生异常
     *
     * @param ctx
     * @param cause
     * @throws Exception
     */
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        System.out.println("MyTextWebSocketFrameHandler.exceptionCaught被调用---异常发生 " + cause.getMessage());
        // 关闭连接
        ctx.close();
    }

    /**
     * 处理客户端数据
     *
     * @param ctx
     * @param msg
     * @throws Exception
     */
    @Override
    protected void channelRead0(ChannelHandlerContext ctx, TextWebSocketFrame msg) throws Exception {
        System.out.println("服务器端收到消息:" + msg.text());
        // 回复消息
        ctx.channel().writeAndFlush(new TextWebSocketFrame("服务器时间" + LocalDateTime.now() + " " + msg.text()));
    }
}
```



## 客户端

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>

<script>
    var socket;
    // 判断当前浏览器是否支持WebSocket编程
    if (window.WebSocket) {
        socket = new WebSocket("ws://localhost:7000/hello");
        // 相当于channelRead0        ev表示收到服务器端回送的消息
        socket.onmessage = function (ev) {
            var rt = document.getElementById("responseText");
            rt.value += "\n" + ev.data;
        };
        // 相当于连接开启（感知到连接开启）
        socket.onopen = function (ev) {
            var rt = document.getElementById("responseText");
            rt.value += "\n连接开启了";
        };
        // 感知到连接关闭
        socket.onclose = function (ev) {
            var rt = document.getElementById("responseText");
            rt.value += "\n连接关闭了";
        };
    } else {
        alert("当前浏览器不支持WebSocket")
    }

    // 发送消息的方法
    function send(message) {
        // 先判断socket是否创建好
        if (!window.socket) {
            return;
        }
        if (socket.readyState == WebSocket.OPEN) {
            // 通过socket发送消息
            socket.send(message);
        } else {
            alert("连接没有开启")
        }
    }
</script>

<form onsubmit="return false">
    <textarea name="message" style="height: 300px;width: 300px;"></textarea>
    <input type="button" value="发送消息" onclick="send(this.form.message.value)">
    <textarea id="responseText" style="width: 300px;height: 300px"></textarea>
    <input type="button" value="清空内容" onclick="document.getElementById('responseText').value=''">
</form>

</body>
</html>
```

