---
title: 06_Shiro_权限URL配置细节
date: 2019-08-29 16:46:42
tags: 
 - Java
 - Shiro
categories:
 - Shiro
---

# 06_Shiro_权限URL配置细节

ShiroFilterFactoryBean的filterChainDefinitions属性；

## 部分细节

- [urls]部分的配置是，其格式是：“**url=拦截器[参数]**，拦截器[参数]”；
- 如果当前请求的url匹配[urls]部分的某个url模式，将会执行其配置的拦截器；
- anon(anonymous)拦截器表示匿名访问（即不需要登录即可访问）；
- authc(authentication)拦截器表示需要身份认证通过后才能访问；



## Shiro中默认的过滤器

![Shiro中默认的过滤器](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/Shiro/06Shiro%E4%B8%AD%E9%BB%98%E8%AE%A4%E7%9A%84%E8%BF%87%E6%BB%A4%E5%99%A8.png)



## URL匹配相关

### URL匹配模式

- **url模式使用Ant风格模式**
- Ant路径通配符支持?、*、**，注意通配符匹配不包括目录分隔符"/"：
  - **？：匹配一个字符**，如/admin?将匹配/admin1，但不匹配/admin或/admin/；
  - ***：匹配零个或多个字符串**，如/admin将匹配/admin、/admin123，但不匹配/admin/1；
  - **\*\*：匹配路径中的零个或多个路径**，如/admin/**将匹配/admin/a或/admin/a/b；



### URL匹配顺序

- **URL权限采取第一次匹配优先的方式**，即从头开始使用第一个匹配的URL模式对应的拦截器链；
- 如：
  - /bb/**=filter1
  - /bb/aa=filter2
  - /**=filter3
  - 如果请求的url是/bb/aa，因为按照声明顺序进行匹配，那么将使用filter1进行拦截；




