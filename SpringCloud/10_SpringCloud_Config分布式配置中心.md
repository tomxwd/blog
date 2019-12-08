---
title: 10_SpringCloud_Config分布式配置中心
date: 2019-12-08 21:13:32
tags: 
 - SpringCloud
categories:
 - SpringCloud
---

# 10_SpringCloud_Config分布式配置中心

## SpringCloud Config配置中心概述

### 分布式系统面临的**配置问题**

微服务意味着要将单个应用中的业务拆分成一个一个子服务，每个服务的粒度相对较小，因此系统中会出现大量的服务。由于每个服务都需要必要的配置信息才能运行，所以一套集中式的、动态的配置管理设施是必不可少的。

SpringCloud提供了Config Server来解决这个问题，我们每一个微服务自己带着一个application.yaml，上百个配置文件的管理......



### SpringCloud Config是什么？

![image-20191208212304136](10_SpringCloud_Config%E5%88%86%E5%B8%83%E5%BC%8F%E9%85%8D%E7%BD%AE%E4%B8%AD%E5%BF%83/image-20191208212304136.png)

SpringCloud Config为微服务架构中的微服务提供集中化的外部配置支持，配置服务器为**各个不同的微服务应用**的所有环境提供了一个**中心化的外部配置**；



### SpringCloud Config怎么玩？

SpringCloud Config分为**服务端和客户端两部分**；

- 服务端：

  服务端也称为**分布式配置中心，它是一个独立的微服务应用**，用于连接配置服务器并且为客户端提供获取配置信息，加密/解密信息等访问接口；

- 客户端：

  客户端则通过指定的配置中心来管理应用资源，以及业务相关的配置内容，并在启动的时候从配置中心获取和加载配置信息；

  配置服务器默认采用git来存储配置信息，这样就有助于对环境配置进行版本管理，并且可以通过git客户端工具来方便的管理和访问配置内容；





SpringCloud Config能干嘛？



SpringCloud Config与GitHub整合配置



## SpringCloud Config服务端配置



## SpringCloud Config客户端配置与测试



## SpringCloud Config配置实战











