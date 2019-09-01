---
title: 08_Docker_阿里云镜像加速器配置
date: 2019-07-07 ‏‎16:19:03
tags: 
 - Docker
categories:
 - Docker
 - Docker初学
---

# 08_Docker_阿里云镜像加速器配置

1. 是什么？

   - [地址](https://dev.aliyun.com/search.html)

2. 注册一个属于自己的阿里云账户（淘宝账户也可以）

3. 获得加速器地址链接

   - 登录阿里云开发者平台
   - 获取加速器地址

4. 配置本机Docker运行镜像加速器

   - 容器镜像服务-------镜像中心-------镜像加速器

     ![镜像加速器地址查看](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/Docker/08%E9%95%9C%E5%83%8F%E5%8A%A0%E9%80%9F%E5%99%A8%E5%9C%B0%E5%9D%80.png)

   - 详细的，阿里云都有写。

   - 鉴于国内网络问题，后续拉取Docker镜像十分缓慢，我们可以需要配置加速器来解决

5. 重新启动Docker后台服务：`serveice docker restart`

6. Linux系统下配置完加速器需要检查是否生效

   - `ps -ef|grep docker`