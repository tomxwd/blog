---
title: 07_Hadoop_编译源码
date: 2019-10-16 22:34:39
tags: 
 - Hadoop
categories:
 - Hadoop
---

# 07_Hadoop_编译源码

## 前期准备工作

**切换到root用户进行操作，减少文件夹权限出现问题**

1. CentOS联网

   配置CentOS能连接外网。

2. jar包准备（hadoop源码、JDK1.8、maven、ant、protobuf）

   - hadoop-2.9.2-src.tar.gz
   - jdk-8u221-linux-x64.tar.gz
   - [](https://archive.apache.org/dist/ant/)（build工具，打包用的，网站中binaries、source中下载）
   - [](https://archive.apache.org/dist/maven/)（maven，网站中找maven3下载）
   - [protobuf](https://github.com/protocolbuffers/protobuf/releases)（序列化的框架）




## tar包安装

**注意切换到root用户操作**

1. JDK解压、配置环境变量JAVA_HOME和PATH，验证java-version
   - 编辑/etc/profile文件
2. MAVEN解压、配置MAVEN_HOME和PATH
3. ant解压、配置ANT_HOME和PATH
4. 安装glibc-headers和g++命令
5. 安装make和cmake
6. 解压protobuf，进入到解压后protobuf主目录，/opt/module/protobuf-2.5.0,然后执行以下命令
7. 安装OpenSSL库
8. 安装ncurses-devel库



## 编译源码

1. 解压源码到/opt/目录

2. 进入到hadoop源码主目录

3. 通过maven执行编译命令

4. 成功的64位hadoop包在/opt/hadoop/hadoop-dist/target下

5. 编译源码过程中常见的问题及解决方案

   - maven install的时候发现JVM内存溢出

     在环境配置文件和maven的执行文件均可调整MAVEN_OPT的heap的大小。

   - 编译期间maven报错。可能网络阻塞问题导致依赖库下载不完整导致，多次执行命令（一次通过比较难）

   - 报ant、protobuf等错误，插件下载未完整或者插件版本问题，最开始链接有较多特殊情况