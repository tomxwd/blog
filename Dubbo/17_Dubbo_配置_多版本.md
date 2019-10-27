---
title: 17_Dubbo_配置_多版本
date: 2019-09-22 10:16:26
tags: 
 - Dubbo
categories:
 - Java
 - Dubbo
---

# 17_Dubbo\_配置_多版本



可以做到灰度发布。

系统服务升级的时候，可以用版本号进行过度，版本号不同的服务相互之间不能互相调用。

打个version属性指定即可。