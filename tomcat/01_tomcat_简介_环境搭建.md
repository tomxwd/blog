---
title: 01_tomcat_简介_环境搭建
date: 2019-10-26 17:25:02
tags: 
 - tomcat
categories:
 - tomcat
---

# 01_tomcat\_简介_环境搭建

> [apache tomcat官网](https://tomcat.apache.org/)
>
> [各个版本下载地址](https://archive.apache.org/dist/tomcat/)

Web服务器主要用来接收客户端发送的请求和响应客户端发送的请求。



## 环境搭建

先要有JAVA_HOME，然后在tomcat的bin下有启动文件。



## 目录结构

| 目录名  | 作用                                                         |
| ------- | ------------------------------------------------------------ |
| bin     | 包含tomcat启动命令、停止tomcat命令等批处理文件，可执行文件   |
| conf    | 配置文件                                                     |
| lib     | tomcat运行依赖的jar包                                        |
| logs    | tomcat的运行日志文件                                         |
| temp    | tomcat的临时文件夹                                           |
| webapps | 里面集合了所有的web项目每一个文件夹就代表一个web项目（默认访问ROOT目录） |
| work    | work保存的tomcat运行时编译好的一些文件                       |



## 修改端口号

conf目录下：

server.xml配置文件，上限是65535



## 对应版本关系

| tomcat版本 | servlet版本 | jsp版本 | el版本 | websocket版本 |
| ---------- | ----------- | ------- | ------ | ------------- |
| tomcat6    | servlet2.5  | jsp2.1  | el     |               |
| tomcat7    | servlet3.0  | jsp2.2  | el2.2  | websocket1.1  |
| tomcat8    | servlet3.1  | jsp2.3  | el3.0  | websocket1.1  |
| tomcat9    | servlet4.0  | jsp2.3  | el3.0  | websocket1.1  |

tomcat6是需要些配置文件的版本，tomcat7之后都是基于注解的，所以tomcat6可以体会一下配置文件来编写web应用。



## eclipse集成tomcat

集成使用的是tomcat的镜像，所以webapps目录下是空的，而且该其配置文件并不会对本机下载的tomcat产生影响。