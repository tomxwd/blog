---
title: 03_用户以及vhost配置
date: 2019-06-13 22:22:11
tags: 
 - Java
 - RabbitMQ
categories:
 - MQ
 - RabbitMQ
---

# 03_用户以及vhost配置

## 添加用户、用户管理

#### 添加用户

点击导航栏的Admin，然后Add a user。

- 用户名
- 密码
- Tags相当于角色，Admin超级管理员、Monitoring、Policymaker、Management、Impersonator、None

![添加用户](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/RabbitMQ/03%E6%B7%BB%E5%8A%A0%E7%94%A8%E6%88%B7.png)

#### virtual host管理

virtual host就相当于mysql的db（数据库）添加virtual host一般以/开头。

点击右侧导航，切换到Virtual Hosts，添加一个新的Virtual Host

![添加新的VirtualHosts](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/RabbitMQ/03%E6%B7%BB%E5%8A%A0VirtualHosts.png)

#### 授权：
点进一个virtual host中有个Permissions栏，Set Permission中选择需要授权的用户。

![授权用户](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/RabbitMQ/03%E6%8E%88%E6%9D%83.png)

#### 控制台的功能介绍

1. Overview : 概览，总共发的信息汇总、监听的端口、使用的协议。
2. Connections : 连接，可以查看有谁连接了
3. Channels : 频道、通道
4. Exchanges : 交换机
5. Queues : 队列
6. Admin : 用户管理、创建Virtual Hosts、授权等