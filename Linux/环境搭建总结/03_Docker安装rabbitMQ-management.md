---
title: 03_Docker安装rabbitMQ-management
date: 2019-11-19 20:05:39
tags: 
 - Linux
categories:
 - Linux
---

# 03_Docker安装rabbitMQ-management

[DockerHub查找RabbitMQ](https://hub.docker.com/_/rabbitmq?tab=description)

安装带有management版本的，有web管理页面

1. 拉取镜像`docker pull rabbitmq:management`

2. 运行容器：

   ```shell
   docker run -d --name my-rabbit -p 12346:5672 -p 12347:15672 -v `pwd`/data:/var/lib/rabbitmq --hostname my-rabbit -e RABBITMQ_DEFAULT_VHOST=my_vhost  -e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=admin
   ```

   - --name：指定容器启动的名字
   - -p：端口映射，一个5672（rabbit服务），一个15672（Web管理控制台）
   - -d：后台运行容器
   - -v：映射目录或文件
   - --hostname：主机名，RabbitMQ的一个重要注意事项，根据所谓的“节点名称”存储数据，默认为主机名
   - -e：指定环境变量
     - RABBIT_DEFAULT_USER：默认的用户名
     - RABBIT_DEFAULT_PASS：默认用户名的密码
     - RABBIT_DEFAULT_VHOST：默认虚拟机名

3. 