---
title: 18_Shiro_多Realm授权的通过标准
date: 2019-08-30 11:24:48
tags: 
 - Java
 - Shiro
categories:
 - Shiro
---

# 18_Shiro_多Realm授权的通过标准

注意到，我们配置多个Realm进行过认证，那么授权使用多Realm的情况下是怎么样的呢？

跟认证器一样，这个多Realm授权器【ModularRealmAuthorizer】也可以进行配置，在源码中可以看到，只要其中一个Realm进行了授权即可；也就是一个通过，即可获得权限；



