---
title: 01_Netty_简介
date: 2019-11-26 23:21:27
tags: 
 - Netty
categories:
 - Netty
---

# 01_Netty_简介

## Netty的介绍

1. Netty是由JBOSS提供的一个**Java开源框架**，现为Github上的独立项目；
2. Netty是一个**异步**的、**基于事件驱动**的网络应用框架，用以快速开发高性能、高可靠性的网络IO程序；
3. Netty主要针对在TCP协议下，面向Clients端的高并发应用，或者Peer-to-Peer场景下的大量数据持续传输的应用。
4. Netty本质是一个NIO框架，适用于服务器通讯相关的多种应用场景；
5. 想透彻理解Netty，需要先学习NIO;
   - **层次结构：TCP/IP=>JDK原生IO/网络=>NIO（IO/网络）=>Netty**



## Netty的应用场景

1. 互联网行业：

   - 在分布式系统中，各个节点之间需要远程服务调用，高性能的RPC框架必不可少，Netty作为异步高性能的通信框架，往往作为基础通信组件被这些RPC框架使用。

   - 典型的应用有：阿里分布式服务框架Dubbo的RPC框架使用Dubbo协议进行节点之间的通信，Dubbo协议默认使用Netty作为基础通信组件，用于实现各进程节点之间的内部通信；

2. 游戏行业：

   - 无论是手游服务端还是大型的网络游戏，Java语言得到了越来越广泛的应用；
   - Netty作为高性能的基础通信组件，提供了TCP/UDP和HTTP协议栈，方便定制和开发私有协议栈，账号登录服务器；
   - 地图服务器之间可以方便的通过Netty进行高性能的通信；

3. 大数据领域

   - 经典的Hadoop的高性能通信和序列化组件（AVRO实现数据文件共享）的RPC框架，默认采用Netty进行跨界点通信；
   - 它的Netty Service基于Netty框架二次封装实现；

4. 其他开源项目使用到Netty

   [看官网介绍；](https://netty.io/wiki/related-projects.html)

   - Akka
   - Flink
   - Spark
   - ......

