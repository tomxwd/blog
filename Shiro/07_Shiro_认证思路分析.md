---
title: 07_Shiro_认证思路分析
date: 2019-08-29 16:51:29
tags: 
 - Java
 - Shiro
categories:
 - Shiro
---

# 07_Shiro_认证思路分析

![Shiro架构(Shiro外部来看)](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/Shiro/07Shiro%E6%9E%B6%E6%9E%84(Shiro%E5%A4%96%E9%83%A8%E6%9D%A5%E7%9C%8B).png)

1. 获取当前的Subject对象，调用SecurityUtils.getSubject()方法；
2. 校验当前的用户是否已经被认证，即是否已经登录，调用Subject的isAuthenticated()方法；
3. 若没有被认证，则把用户名和密码封装成UsernamePasswordToken对象；
   - 创建一个表单页面
   - 把请求提交到springmvc的handler
   - 获取用户名密码
4. 执行登录：调用Subject的login(AuthenticationToken【接口】)方法来执行登录；
5. 自定义Realm的方法，从数据库中获取对应的记录，返回给Shiro。
   - 实际上需要继承org.apache.shiro.realm.AuthenticatingRealm类。
   - 实现doGetAuthenticationInfo(AuthenticationToken)方法。
6. 由shiro完成对密码的比对。

这里的第五步，为什么要继承这个AuthenticatingRealm类而不是其他呢？

如果是要实现认证功能，需要继承这个类；

因为从源码可以看到最终调用的是doGetAuthenticationInfo(AuthenticationToken)方法来进行认证。

