---
title: 01_Docker安装
date: 2019-11-18 21:41:45
tags: 
 - Linux
categories:
 - Linux
---

# 01_Docker安装

1. `yum install -y epel-release`

2. `yum install -y docker-io`

3. `vim /etc/docker/daemon.json`

   ```json
   {
     "registry-mirrors": ["到阿里云控制台找容器镜像服务-镜像加速器"]
   }
   ```

   `systemctl daemon-reload`

   `systemctl restart docker`

4. 

   - CentOS6：`service docker start`

   - CentOS7：`systemctl start docker`

5. `docker version`验证是否安装成功

6. `docker info`查看镜像是否生效