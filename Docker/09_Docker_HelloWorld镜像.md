---
title: 09_Docker_HelloWorld镜像
date: 2019-07-07 ‏‎16:33:56
tags: 
 - Docker
categories:
 - Docker
 - Docker初学
---

# 09_Docker_HelloWorld镜像

启动Docker后台容器（测试运行hello-world）

1. `docker run hello-world`
   - 由于本地没有hello-world镜像，所以会先去拉取一个hello-world镜像，并在容器内运行。
   - 输出提示之后，hello-world会停止运行，容器就会停止。
2. run干了什么？
   - 首先会现在本机中找是否存在该镜像
     - 存在该镜像
       - 以该镜像为模板生产容器实例运行
     - 不存在该镜像
       - 去DockerHub仓库上搜该镜像是否存在
         - DockerHub不存在该镜像
           - 返回失败错误信息，找不到该镜像
         - DockerHub存在该镜像
           - 下载该镜像到本地
           - 以该镜像为模板生产容器实例运行