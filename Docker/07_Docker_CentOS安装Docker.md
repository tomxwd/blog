---
title: 07_Docker_CentOS安装Docker
date: 2019-07-07 16:04:56
tags: 
 - Docker
categories:
 - Docker
 - Docker初学
---

# 07_Docker_CentOS安装Docker

> CentOS   6.8   安装Docker
>
> [官网文档](https://docs.docker.com/install/linux/docker-ce/centos/)

## CentOS 6安装步骤：

1. `yum install -y epel-release`
   - Docker是使用EPEL发布，RHEL系的OS首先要确保已经持有EPEL仓库，否则先检查OS的版本，然后安装相应的EPEL包。
2. `yum install -y docker-io`
3. 安装后的配置文件：`/etc/sysconfig/docker`
4. 启动Docker后台服务：`service docker start`
5. `docker version`验证是否安装成功

 

