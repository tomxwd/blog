---
title: 06_Dubbo_环境搭建_Zookeeper注册中心
date: 2019-09-08 17:21:02
tags: 
 - Dubbo
categories:
 - Java
 - Dubbo
---

# 06_Dubbo\_环境搭建_Zookeeper注册中心

[Zookeeper](http://zookeeper.apache.org/) 是 Apacahe Hadoop 的子项目，是一个树型的目录服务，支持变更推送，适合作为 Dubbo 服务的注册中心，工业强度较高，可用于生产环境，并推荐使用。

Dubbo支持很多种注册中心，官方建议是使用zookeeper作为注册中心。

> [zookeeper官网](http://zookeeper.apache.org/)
>
> [zookeeper下载地址](https://archive.apache.org/dist/zookeeper/)

![1567935115883](06_Dubbo_环境搭建_Zookeeper注册中心/1567935115883.png)

## windows下

1. bin目录下的可执行文件（zkServer.cmd），点击运行，发现报错，没有找到zoo.cfg文件。
2. 在conf目录下有个zoo_sample.cfg，复制一份，内容：
   - clientPort=2181【默认端口2181】
   - dataDir=/tmp/zookeeper【临时数据存放位置】
3. 启动起zkServer之后，用bin目录下的zkCli.cmd进行连接

