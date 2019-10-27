---
title: 24_Dubbo_原理_RPC&Netty原理
date: 2019-09-22 16:37:07
tags: 
 - Dubbo
categories:
 - Java
 - Dubbo
---

# 24_Dubbo\_原理_RPC&Netty原理

## RPC原理

![1569141680902](24_Dubbo_%E5%8E%9F%E7%90%86_/1569141680902.png)

一次完整的RPC调用流程（同步调用，异步另说），如下：

1. 服务消费方(client)调用以本地调用方式调用服务；
2. client stub接收到调用后负责将方法、参数等组装成能够进行网络传输的消息体；
3. client stub找到服务地址，并将消息发送到服务端；
4. server stub收到消息后进行解码；
5. server stub根据解码结果调用本地的服务；
6. 本地服务执行并将结果返回给server stub；
7. server stub将返回结果打包成消息并发送至消费方；
8. client stub接收到消息，并进行解码；
9. 服务消费方得到最终结果；

RPC框架的目标就是要将上面2~8的步骤封装起来，这些细节对于用户来说是透明的，不可见的。

我们用dubbo框架的时候，只是使用了第一步和第二步，因为2~8是dubbo帮我们处理的。



## netty通信原理

Netty是一个异步事件驱动的网络应用程序框架，用于快速开发可维护的高性能协议服务器和客户端。它极大地简化了TCP和UDP套接字服务器等网络编程：

BIO（BlockingIO【阻塞式IO】）：

![1569141888189](24_Dubbo_%E5%8E%9F%E7%90%86_/1569141888189.png)

NIO（Non-BlockingIO【非阻塞式IO】）：

![1569142066849](24_Dubbo_%E5%8E%9F%E7%90%86_/1569142066849.png)

Selector一般称为选择器，也可以翻译为多路复用器；

- Connect（连接就绪）
- Accept（接受就绪）
- Read（读就绪）
- Write（写就绪）



### netty基本原理

![1569142182076](24_Dubbo_%E5%8E%9F%E7%90%86_/1569142182076.png)