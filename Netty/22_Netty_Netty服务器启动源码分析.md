---
title: 22_Netty_Netty服务器启动源码分析
date: 2019-12-28 10:34:29
tags: 
 - Netty
categories:
 - Netty
---

# 22_Netty_Netty服务器启动源码分析

**基本说明：**

1. 只有看过Netty源码，才能说真正掌握了Netty框架；
2. 在io.netty.example包下，有很多Netty源码案例，可以用来分析；

**源码剖析目的：**用源码分析的方式走一下Netty（服务器）的启动过程，更好的理解Netty的整体设计和运行机制；

**说明：**

1. 源码需要剖析到Netty调用的doBind方法，追踪到NioServerSocketChannel的doBind
2. 并且要Debug程序到NioEventLoop类的**run代码，无限循环**，在服务器端运行；











