---
title: 02_WebSocket_API介绍
date: 2019-11-25 22:34:36
tags: 
 - WebSocket
categories:
 - WebSocket
---

# 02_WebSocket_API介绍

实现一个WebSocket应用程序需要如下步骤：

1. 开启连接
2. 客户端给服务器端发送数据
3. 服务器端接收数据
4. 服务器端给客户端发送数据
5. 客户端接收数据
6. 监听三类基本事件：
   - onopen：打开连接时的响应事件
   - onmessage：发送数据时的响应事件
   - onclose：关闭连接时的响应事件

