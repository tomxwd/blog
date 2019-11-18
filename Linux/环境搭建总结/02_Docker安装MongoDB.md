---
title: 02_Docker安装MongoDB
date: 2019-11-18 21:59:58
tags: 
 - Linux
categories:
 - Linux
---

# 02_Docker安装MongoDB

[docker官网查MongoDB](https://hub.docker.com/_/mongo)

[菜鸟教程-Docker安装MongoDB教程](https://www.runoob.com/docker/docker-install-mongodb.html)

上面详细的启动帮助文档等。

1. 直接拉取最新版本：`docker pull mongo:latest`
2. 用`docker images`可以查看镜像信息
3. 运行容器`docker run --name my-mongo -p 12345:27017 -itd mongo --auth`
   - --name：指定容器别名
   - -p：指定端口映射
   - -d：后台运行，并返回容器id
   - -it：t是为容器重新分配一个伪输入终端，通常和i一起使用，而i是以交互模式运行容器
   - --auth：需要密码才能访问MongoDB容器
4. `docker ps`查看容器运行情况
5. 进入容器：`docker exec -it my-mongo /bin/bash`
6. 进入mongo：`mongo`
7. 切换到admin DataBase：`use admin`
8. 也可以直接用`docker exec -it my-mongo mongo admin`来直接进入mongo并使用amdin用户。
9. 创建用户：`db.createUser({user:'root',pwd:'root',roles:[{role:'userAdminAnyDatabase',db:'admin'}]});`
10. 可以用`show users`查看当前database用户。
11. 创建一个普通用户，对foo数据库有读写权限：`use foo
    switched to db foo db.createUser({
    user:"tomxwd",pwd:"tomxwd",roles:[{role:"readWrite",db:"foo"}]})`
12. 用普通用户进行登录并操作foo数据库即可。