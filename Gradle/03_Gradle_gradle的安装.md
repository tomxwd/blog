---
title: 03_Gradle_gradle的安装
date: 2019-08-27 11:09:46
tags: 
 - gradle
categories:
 - Java
 - gradle
---

# 03_Gradle_gradle的安装

> [gradle官网](https://gradle.org/)
>
> [gradle下载地址](http://services.gradle.org/distributions/)
>
> 下载bin.zip

下载好后解压，进行环境变量配置：

| 变量        | 值                         |
| ----------- | -------------------------- |
| GRADLE_HOME | D:\environment\gradle-5.6; |
| Path        | %GRADLE_HOME%\bin;         |

在cmd中输入`gradle -v`，出现版本信息则安装、配置成功。