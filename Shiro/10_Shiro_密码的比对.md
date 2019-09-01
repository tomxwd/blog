---
title: 10_Shiro_密码的比对
date: 2019-08-30 08:40:25
tags: 
 - Java
 - Shiro
categories:
 - Shiro
---

# 10_Shiro_密码的比对

## 登录流程：

用户登录输入用户名密码，发出登录请求，由SecurityUtils.getSubject()获取Subject对象，再调用Subject对象的login方法，此时会调用Realm的doGetAuthenticationInfo()方法进行验证。



## DEBUG流程：

我们知道，Shiro进行比对，那肯定会调用到UsernamePasswordToken的getPassword()方法，那么我们在这个方法上打个断点，再运行程序，就可以往前追溯到调用他的地方。

可以发现是通过AuthenticatingRealm的credentialsMatcher属性来进行密码的比对；

那么就可以发现，Shiro支持自定义或者使用实现好的credentialsMatcher进行密码校验；