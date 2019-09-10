---
title: 03_Dubbo_概念_RPC简介
date: 2019-09-08 15:08:03
tags: 
 - Dubbo
categories:
 - Java
 - Dubbo
---

# 03_Dubbo\_概念_RPC简介

## RPC

### 什么叫RPC

RPC【Remote Procedure Call】是指远程过程调用，是一种进程间通信方式，他是一种技术思想，而不是规范。它允许程序调用另一个地址空间（通常是共享网络的另一台机器上）的过程或函数，而不是程序员显式编码这个远程调用的细节。即程序员无论是调用本地还是远程的函数，本质上编写的调用代码基本相同。

### RPC基本原理

![1567932088699](03_Dubbo_概念_RPC简介/1567932088699.png)

下面是一个例子

```sequence
participant Client
participant Client Stub
participant Server Stub
participant Server

Client->Client Stub:1.客户端调用
Client Stub->Client Stub:2.序列化
Client Stub->Server Stub:3.发送信息
Server Stub->Server Stub:4.反序列化
Server Stub->Server:5.调用本地服务
Server->Server:6.服务处理
Server-->Server Stub:7.返回处理结果
Server Stub->Server Stub:8.将结果序列化
Server Stub-->Client Stub:9.返回消息
Client Stub->Client Stub:10.反序列化
Client Stub->Client:11.返回调用结果
```

RPC两个核心模块：通讯、序列化。

RPC框架有很多如：

Dubbo、gRPC、Thrift、HSF（High Speed Service Framework）

