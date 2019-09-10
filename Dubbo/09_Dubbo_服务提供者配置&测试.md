---
title: 09_Dubbo_服务提供者配置&测试
date: 2019-09-09 22:11:06
tags: 
 - Dubbo
categories:
 - Java
 - Dubbo
---

# 09_Dubbo_服务提供者配置&测试

步骤：

1. 将服务提供者注册到注册中心（暴露服务）

   - 导入dubbo依赖（2.6.2）

     ```xml
     <!-- dubbo -->
     <dependency>
         <groupId>com.alibaba</groupId>
         <artifactId>dubbo</artifactId>
         <version>2.6.2</version>
     </dependency>
     ```

   - 引入操作Zookeeper的客户端

     2.6以前引入zkClient：

     ```xml
     <dependency>
         <groupId>com.101tec</groupId>
         <artifactId>zkclient</artifactId>
         <version>0.10</version>
     </dependency>
     ```

     2.6以后引入curator-framework:

     ```xml
     <dependency>
         <groupId>org.apache.curator</groupId>
         <artifactId>curator-framework</artifactId>
         <version>2.12.0</version>
     </dependency>
     ```

     

2. 让服务消费者去注册中心订阅服务提供者的服务地址