---
title: 26_Shiro_认证和记住我的区别
date: 2019-08-30 16:17:57
tags: 
 - Java
 - Shiro
categories:
 - Shiro
---

# 26_Shiro_认证和记住我的区别

## 概述

- Shiro提供了记住我（RememberMe）的功能，比如访问淘宝等一些网站的时候，关闭了浏览器，下次再打开还是能记住你是谁，下次访问时就无需再登录即可访问，基本流程如下：
  1. 首先在登录页面选中RememberMe然后登录成功；如果是浏览器登录，一般会把**RememberMe的Cookie写到客户端并保存下来**；
  2. 关闭浏览器再重新打开；会发现浏览器还是记住你的；
  3. 访问一般的网页服务器端还是知道你是谁，且能正常访问；
  4. 但是比如我们访问淘宝时，如果要**查看我的订单或者进行支付的时候，此时还是需要再进行身份认证的，以确保当前用户还是你**；



## 认证和记住我

- subject.isAuthenticated()表示用户进行了**身份验证登录**的，即是有**Subject.login()**进行了登录；
- subject.isRemembered()表示用户是通过**记住我**登录的，这时候可能不是真正的你（如你的朋友使用你的电脑，或者你的cookie被窃取）在访问的；
- **二者二选一**，即subject.isAuthenticated()==true，则subject.isRemembered()==false；反之一样。



## 建议

- **访问一般网页**：如个人在主页之类的，我们使用user拦截器即可，user拦截器只要用户登录，通过`(isRemembered()||isAuthenticated())`过即可访问成功；
- **访问特殊网页**：如我的订单，提交订单页面，我们使用authc拦截器即可，authc拦截器会判断用户是否是通过Subject.login()(isAuthenticated()==true)登录的，如果是才放行，否则会跳转到登录页面叫你重新登录。



