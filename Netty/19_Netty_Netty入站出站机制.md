---
title: 19_Netty_Netty入站出站机制
date: 2019-12-23 21:33:51
tags: 
 - Netty
categories:
 - Netty
---

# 19_Netty_Netty入站出站机制

1. netty的组件设计：

   - Channel
   - EventLoop
   - ChannelFuture
   - ChannelHandler
   - ChannelPipeline

2. ChannelHandler充当了处理入站和出站数据的应用程序逻辑的容器。例如，实现ChannelInboundHandler接口（或者ChannelInboundHandlerAdapter），你就可以接收入站事件和数据，这些数据会被业务逻辑处理。当要给客户端发送响应的时候，也可以从ChannelInboundHandler冲刷数据。业务逻辑通常写在一个或多个ChannelInboundHandler中。ChannelOutboundHandler原理一样，只不过他是用来处理出站数据的；

3. ChannelPipeline提供了ChannelHandler链的容器。以客户端应用程序为例，如果事件方向是从客户端到服务器的，那么我们称这些事件为出站的，即客户端发送给服务端的数据会通过pipeline中的一系列ChannelOutboundHandler，并被这些Handler处理，反之则称之为入站事件；

   ![img](19_Netty_Netty%E5%85%A5%E7%AB%99%E5%87%BA%E7%AB%99%E6%9C%BA%E5%88%B6/u=1421918122,2661459927&fm=26&gp=0.jpg)

   可以看做左边是服务器，右边是客户端；



## 编码解码器

1. 当Netty发送或者接受一个消息的时候，就将会发生一次数据转换。**入站消息会被解码，从字节转换为另一种格式（比如java对象）；如果是出站消息，会被编码为字节**；
2. Netty提供一系列实用的编解码器，他们都实现了ChannelInboundHandler或者ChannelOutboundHandler接口。在这些类中，channelRead方法已经被重写了。以入站为例，对于每个从入站Channel读取的消息，这个方法会被调用。随后，它将调用由解码器所提供的decode()方法进行解码，并将已经解码的字节转发给ChannelPipeline中的下一个ChannelInboundHandler。



### 解码器-ByteToMessageDecoder

1. 继承图

   ![image-20191223220242794](19_Netty_Netty%E5%85%A5%E7%AB%99%E5%87%BA%E7%AB%99%E6%9C%BA%E5%88%B6/image-20191223220242794.png)

2. 由于不可能知道远程节点是否会一次性发送一个完整的信息，tcp有可能出现粘包拆包的问题，这个类会对入站数据进行缓冲，直到他准备好被处理；

3. 一个关于ByteToMessageDecoder实例分析

   ```java
   public class ToIntegerDecoder extends ByteToMessageDecoder {
       @Override
       protected void decode(ChannelHandlerContext ctx,ByteBuf in,List<Object> out) throws Exception{
           if(in.readableBytes()>=4){
               out.add(in.readInt());
           }
       }
   }
   ```

   说明：

   - 这个例子，每次入站从ByteBuf中读取4个字节，将其解码为一个int，然后将他添加到下一个List中。
   - 当没有更多元素可以被添加到该List中的时候，它的内容将会发送给下一个ChannelInboundHandler，int在被添加到List中的时候，会被自动装箱为Integer。
   - 在调用readInt()方法钱必须验证所输入的ByteBuf是否具有足够的数据；

   ![image-20191223221830307](19_Netty_Netty%E5%85%A5%E7%AB%99%E5%87%BA%E7%AB%99%E6%9C%BA%E5%88%B6/image-20191223221830307.png)



## Handler链调用机制实例

实例要求：

![image-20191225231249695](19_Netty_Netty%E5%85%A5%E7%AB%99%E5%87%BA%E7%AB%99%E6%9C%BA%E5%88%B6/image-20191225231249695.png)

使用自定义的编码器和解码器来说明Netty的Handler调用机制

客户端发送long->服务器

服务器发送long->客户端



代码：

编解码器：

```java
/**
 * 解码器byte->long
 *
 * @author xieweidu
 * @createDate 2019-12-25 21:07
 */
public class MyByteToLongDecoder extends ByteToMessageDecoder {
    /**
     * decode会根据接收的数据，被调用多次，直到确定没有新的元素被添加到list，或者ByteBuf没有更多的可读字节为止
     * 如果list out不为空，就会将list的内容传递给下一个ChannelInboundHandler处理，该处理器的方法也会被调用多次
     * @param ctx 上下文对象
     * @param in  入站的ByteBuf
     * @param out List集合，将解码后的数据传给下一个Handler
     * @throws Exception
     */
    @Override
    protected void decode(ChannelHandlerContext ctx, ByteBuf in, List<Object> out) throws Exception {
        System.out.println("MyByteToLong decoder方法被调用");

        // 因为long是8个字节，需要判断有8个才读取
        if (in.readableBytes() >= 8) {
            out.add(in.readLong());
        }
    }
}
```

```java
/**
 * 编码器
 * @author xieweidu
 * @createDate 2019-12-25 21:23
 */
public class MyLongToByteEncoder extends MessageToByteEncoder<Long> {

    /**
     * 编码方法
     *
     * @param ctx
     * @param msg
     * @param out
     * @throws Exception
     */
    @Override
    protected void encode(ChannelHandlerContext ctx, Long msg, ByteBuf out) throws Exception {
        System.out.println("MyLongToByteEncoder encode方法被调用");
        System.out.println("msg=" + msg);
        out.writeLong(msg);
    }
}
```



服务器端：

```java
public class MyServer {

    public static void main(String[] args) {
        NioEventLoopGroup bossGroup = new NioEventLoopGroup(1);
        NioEventLoopGroup workerGroup = new NioEventLoopGroup(8);
        try {
            ServerBootstrap serverBootstrap = new ServerBootstrap();
            serverBootstrap.group(bossGroup, workerGroup)
                    .channel(NioServerSocketChannel.class)
                    .childHandler(new MyServerInitializer());
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

```java
public class MyServerHandler extends SimpleChannelInboundHandler<Long> {

    @Override
    protected void channelRead0(ChannelHandlerContext ctx, Long msg) throws Exception {
        System.out.println("从客户端:" + ctx.channel().remoteAddress() + "读取到long:" + msg);
        // 给客户端回送一个Long信息
        ctx.writeAndFlush(987654L);
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        cause.printStackTrace();
        ctx.close();
    }
}
```

```java
public class MyServerInitializer extends ChannelInitializer<SocketChannel> {

    @Override
    protected void initChannel(SocketChannel ch) throws Exception {
        ChannelPipeline pipeline = ch.pipeline();
        // 对于服务器来说，客户端发送long到服务器，入站操作，需要解码，
        pipeline.addLast(new MyByteToLongDecoder())
                .addLast(new MyLongToByteEncoder())
                .addLast(new MyServerHandler());
    }
}
```

客户端：

```java
public class MyClient {

    public static void main(String[] args) {
        NioEventLoopGroup group = new NioEventLoopGroup(1);
        try {
            Bootstrap bootstrap = new Bootstrap();
            bootstrap.group(group)
                    .channel(NioSocketChannel.class)
                    .handler(new MyClientInitializer());
            ChannelFuture channelFuture = bootstrap.connect("localhost", 7000).sync();
            channelFuture.channel().closeFuture().sync();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            group.shutdownGracefully();
        }

    }

}
```

```java
public class MyClientHandler extends SimpleChannelInboundHandler<Long> {

    @Override
    protected void channelRead0(ChannelHandlerContext ctx, Long msg) throws Exception {

        System.out.println("服务器的ip：" + ctx.channel().remoteAddress());
        System.out.println("收到服务器消息：" + msg);

    }

    /**
     * 重写channelActive发送消息
     *
     * @param ctx
     * @throws Exception
     */
    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        System.out.println("MyClientHandler发送数据");
        // 发送long
        ctx.writeAndFlush(123456L);

        /**
         * 分析：
         * 1. abcdabcdabcdabcd有16个字节，发送的时候会以8个为一组
         * 2. 该处理器的后一个handler是MyLongToByteEncoder
         * 3. 父类MessageToByteEncoder有个write方法：
         *     @Override
         *     public void write(ChannelHandlerContext ctx, Object msg, ChannelPromise promise) throws Exception {
         *         ByteBuf buf = null;
         *         try {
         *             if (acceptOutboundMessage(msg)) {
         *                 @SuppressWarnings("unchecked")
         *                 I cast = (I) msg;
         *                 buf = allocateBuffer(ctx, cast, preferDirect);
         *                 try {
         *                     encode(ctx, cast, buf);
         *                 } finally {
         *                     ReferenceCountUtil.release(cast);
         *                 }
         *
         *                 if (buf.isReadable()) {
         *                     ctx.write(buf, promise);
         *                 } else {
         *                     buf.release();
         *                     ctx.write(Unpooled.EMPTY_BUFFER, promise);
         *                 }
         *                 buf = null;
         *             } else {
         *                 ctx.write(msg, promise);
         *             }
         *         } catch (EncoderException e) {
         *             throw e;
         *         } catch (Throwable e) {
         *             throw new EncoderException(e);
         *         } finally {
         *             if (buf != null) {
         *                 buf.release();
         *             }
         *         }
         *     }
         * 判断了如果类型匹配不上，就不做处理直接write
         * 4. 因此编写Encoder的时候要注意传入的数据类型和处理的数据类型要一致
         */
//        ctx.writeAndFlush(Unpooled.copiedBuffer("abcdabcdabcdabcd", CharsetUtil.UTF_8));
    }
}
```

```java
public class MyClientInitializer extends ChannelInitializer<SocketChannel> {
    @Override
    protected void initChannel(SocketChannel ch) throws Exception {
        ChannelPipeline pipeline = ch.pipeline();
        // 出站加入一个Handler进行编码
        pipeline.addLast(new MyLongToByteEncoder())
                .addLast(new MyByteToLongDecoder())
                .addLast(new MyClientHandler());
    }
}
```



结论：

1. 不论解码器Handler还是编码器Handler，接收到的消息类型必须与待处理的消息类型一致，否则该Handler不会被执行。
2. 在解码器进行数据解码的时候，需要判断缓冲区（ByteBuf）的数据是否足够，否则接收到的结果和期望的结果可能不一致。



## 解码器-ReplayingDecoder

1. public abstract class ReplayingDecoder\<S> extends ByteToMessageDecoder

2. ReplayingDecoder扩展了ByteToMessageDecoder类，使用这个类，我们不必调用readableBytes()方法。参数S指定了用户状态管理的类型，其中Void表示不需要状态管理；

3. 应用实例：使用ReplayingDecoder编写解码器，对前面的案例进行优化；

   ```java
   public class MyByteToLongDecoder2 extends ReplayingDecoder<Void> {
       @Override
       protected void decode(ChannelHandlerContext ctx, ByteBuf in, List<Object> out) throws Exception {
           System.out.println("MyByteToLongDecoder2 被调用");
           // 在ReplayingDecoder不需要判断是否足够读取，内部会进行判断
           out.add(in.readLong());
       }
   }
   ```

4. ReplayingDecoder使用方便，但是有一定的局限性：

   - 并不是所有的ByteBuf操作都被支持，如果调用了一个不被支持的方法，将会抛出UnsupportedOperationException异常；
   - ReplayingDecoder在某些情况下可能稍微慢于ByteToMessageDecoder，例如网络缓慢且消息格式复杂，消息会被拆分为多个碎片，速度会变慢；



## 其他编解码器

解码器：

1. LineBasedFrameDecoder：这个类在Netty内部也有使用，使用行尾控制字符(\n或者\r\n)作为分隔符来解析数据；
2. DelimiterBasedFrameDecoder：使用自定义的特殊字符作为消息的分隔符；
3. HttpObjectDecoder：一个Http数据的解码器；
4. LengthFieldBasedFrameDecoder：通过指定长度来标识整包消息，这样就可以自动的处理粘包和半包的消息；

编码器与之对应；