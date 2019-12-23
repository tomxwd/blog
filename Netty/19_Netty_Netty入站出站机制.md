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



## Handler链调用机制实例1





