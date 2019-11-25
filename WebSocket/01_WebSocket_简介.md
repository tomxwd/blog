---
title: 01_WebSocket_简介
date: 2019-11-25 21:56:22
tags: 
 - WebSocket
categories:
 - WebSocket
---

# 01_WebSocket_简介

- WebSocket目标：打破传统的Web请求响应模型，实现**管道式**的实时通信；

- 打开一个浏览器和服务器的通信管道，持续连接；
- 服务器给浏览器推送数据非常方便；
- Web的实时消息通信：聊天、股票、游戏、监控等；

从Tomcat7开始就支持WebSocket，支持最新的WebSocket开发规范JSR356；

可以运行Tomcat查看Example，里面有用WebSocket实现的小案例；

由于在Tomcat的lib文件夹下多了个WebSocket-api.jar（接口），且有个tomcat7-websocket.jar（实现），tomcat7-websocket即为实现，所以Tomcat7支持WebSocket；



## WebSocket API介绍

- ServerApplicationConfig

  项目启动时会自动启动，类似于ContextListener，是WebSocket的核心配置类

  两个方法：

  - getEndPointConfigs：获取所有以接口方式配置的WebSocket类
  - getAnnotatedEndpointClassess：扫描src下所有@ServerEndPoint注解的类；
    - EndPoint就指的是一个WebSocket的一个服务端程序（相当于Servlet）

实现这个接口，那么就会在项目启动的时候自动执行；



1. 创建一个Web项目（Tomcat+maven）

2. 创建一个类实现ServerApplicationConfig接口，这里用注解的形式来使用，所以getEndPointConfigs(实现接口形式)方法体则留空即可；

   ```java
   public class WebSocketConfig implements ServerApplicationConfig {
       // 接口方式编程
       @Override
       public Set<ServerEndpointConfig> getEndpointConfigs(Set<Class<? extends Endpoint>> set) {
           return null;
       }
   
       // 注解方式编程
       @Override
       public Set<Class<?>> getAnnotatedEndpointClasses(Set<Class<?>> set) {
           System.out.println("启动");
           System.out.println("set = " + set);
           // 返回需要注册的Socket类，此处可以做过滤
           return set;
       }
   }
   ```

3. 创建一个类，在类上加@ServerEndpoint("/url")注解

   ```java
   @ServerEndpoint("/echo")
   public class EchoSocket {
   }
   ```

4. 启动项目，可以看到打印了一个对象；

