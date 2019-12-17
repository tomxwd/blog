---
title: 07_Netty_Netty概述
date: 2019-12-02 20:59:26
tags: 
 - Netty
categories:
 - Netty
---

# 07_Netty_Netty概述

## 原生NIO存在的问题

1. NIO的类库和API繁杂，使用麻烦：需要熟练掌握Selector、ServerSocketChannel、SocketChanel、ByteBuffer等；
2. 需要具备其他的额外技能：熟悉Java多线程编程，因为NIO编程设计到Reactor模式，必须要对多线程和网络编程非常熟悉，才能编写出高质量的NIO程序；
3. 开发工作量和难度都非常大：例如客户端面临断连重连、网络闪断、半包读写、失败缓存、网络阻塞和异常流的处理等等；
4. JDK NIO 的Bug：例如臭名昭著的Epoll Bug，会导致Selector空轮询，最终导致CPU 100%占用。直到JDK1.7版本该问题仍旧存在，没有被根本解决；



## Netty官网说明

[Netty官网](https://netty.io/)

 Netty is *an asynchronous event-driven network application framework*
for rapid development of maintainable high performance protocol servers & clients. 

Netty是一个异步的基于事件驱动的网络应用框架，为了快速开发服务器端和客户端；

 ![netty官网组件介绍](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/Netty/07netty%E5%AE%98%E7%BD%91%E7%BB%84%E4%BB%B6%E4%BB%8B%E7%BB%8D.png)

1. Netty是由JBOSS提供的一个Java开源框架。
2. Netty提供异步的、基于事件驱动是网络应用程序框架，用以快速开发高性能、高可靠性的网络IO程序；
3. Netty可以帮助你快速、简单的开发出一个网络应用，相当于简化和流程化了NIO的开发过程；
4. Netty是目前最流行的NIO框架，Netty在互联网领域、大数据分布式计算领域、游戏行业、通信行业等获得了广泛的应用，知名的Elasticsearch、Dubbo框架内部都采用了Netty；



## Netty的优点

Netty对JDK自带的NIO的API进行了封装；

1. 设计优雅：
   - 适用于各种传输类型的统一API阻塞和非阻塞Socket；
   - 基于灵活且可扩展的事件模型，可以清晰地分离关注点；
   - 高度可定制的线程模型——单线程，一个或多个线程池；
2. 使用方便：详细记录的JavaDoc，用户指南和示例；没有其他依赖项，JDK 5 （Netty 3.x）或6（Netty 4.x）就足够了；
3. 高性能、吞吐量更高、延迟更低、减少资源消耗、最小化不必要的内存复制；
4. 安全：完整的SSL/TLS和StartTLS支持；
5. 社区活跃、不断更新、版本迭代周期端、发现的Bug可以被及时修复，同时又更多的新功能被加入；



## Netty版本说明

1. Netty版本分为netty3.x和netty4.x、netty5.x
2. 因为Netty5出现重大Bug，被官网废弃了，目前推荐使用的是Netty4.x的稳定版；
3. 目前在官网可以下载到netty3.x、netty4.x、netty4.1.x；
4. 这里我们使用netty4.1.x版本；
5. [netty官网下载页](https://netty.io/downloads.html)、[netty下载地址](https://bintray.com/netty/downloads/netty)





