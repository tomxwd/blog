---
title: 03_Docker_理念
data: 2019-07-06 23:18:28
tags: 
 - Docker
categories:
 - Docker
 - Docker初学
---

# 03_Docker_理念

Docker是基于**Go语言实现的云开源项目**。

Docker的主要目标是"Buid, Ship and Run Any App, Anywhere"，也就是通过对应用组件的封装、分发、部署、运行等生命周期的管理，使用户的APP（可以是一个WEB应用或数据库应用等）及其运行环境能够做到“一次封装，到处运行”。

![Docker理念](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/Docker/03Docker%E7%90%86%E5%BF%B5.png)

Linux容器技术的出现就解决了这样一个问题，而Docker就是在他的基础上发展过来的。将应用运行在Docker容器上面，而Docker容器在任何操作系统上都是一致的，这就实现了**跨平台、跨服务器。**

只需要一次配置好环境，换到别的机子就可以一键部署好，大大简化了操作。