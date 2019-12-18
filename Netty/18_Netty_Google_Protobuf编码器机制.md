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

## 