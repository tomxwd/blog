---
title: 01_IDEA_创建maven-web工程配置
date: 2019-08-28 17:12:08
tags: 
 - IDEA
categories:
 - IDEA
 - IDEA使用
---

# 01_IDEA_创建maven-web工程配置

## 选择maven模板方式创建

![01_IDEA创建web-01](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/IDEA/%E4%BD%BF%E7%94%A8/01_IDEA%E5%88%9B%E5%BB%BAweb-01.png)

创建两个文件夹：

![01_IDEA创建web-02](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/IDEA/%E4%BD%BF%E7%94%A8/01_IDEA%E5%88%9B%E5%BB%BAweb-02.png)

添加资源路径：

![01_IDEA创建web-03](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/IDEA/%E4%BD%BF%E7%94%A8/01_IDEA%E5%88%9B%E5%BB%BAweb-03.png)



## 不选择任何模板创建

创建普通maven项目目录结构：

![image-20200328110543034](01_IDEA_%E5%88%9B%E5%BB%BAmaven-web%E5%B7%A5%E7%A8%8B%E9%85%8D%E7%BD%AE/image-20200328110543034.png)

CTRL+SHIFT+ALT+S打开项目配置 --> 找到Modules点击+号新增框架支持

![image-20200328110758367](01_IDEA_%E5%88%9B%E5%BB%BAmaven-web%E5%B7%A5%E7%A8%8B%E9%85%8D%E7%BD%AE/image-20200328110758367.png)

按默认流程走即可：

![image-20200328112457763](01_IDEA_%E5%88%9B%E5%BB%BAmaven-web%E5%B7%A5%E7%A8%8B%E9%85%8D%E7%BD%AE/image-20200328112457763.png)

添加tomcat服务启动，前提是安装了tomcat且配置了tomcat信息（项目配置主要就配置上下文路径）：

![image-20200328112509250](01_IDEA_%E5%88%9B%E5%BB%BAmaven-web%E5%B7%A5%E7%A8%8B%E9%85%8D%E7%BD%AE/image-20200328112509250.png)

构建完成项目目录结构：

![image-20200328112557683](01_IDEA_%E5%88%9B%E5%BB%BAmaven-web%E5%B7%A5%E7%A8%8B%E9%85%8D%E7%BD%AE/image-20200328112557683.png)

