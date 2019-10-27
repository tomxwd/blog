---
title: 18_Dubbo_配置_本地存根
date: 2019-09-22 10:29:33
tags: 
 - Dubbo
categories:
 - Java
 - Dubbo
---

# 18_Dubbo\_配置_本地存根

![/user-guide/images/stub.jpg](18_Dubbo_%E9%85%8D%E7%BD%AE_%E6%9C%AC%E5%9C%B0%E5%AD%98%E6%A0%B9/stub.jpg)

消费方一般只需要接口，其实现全在提供方（服务器端），但是提供方有时候想在消费方这边也执行部分逻辑，比如：做ThreadLocal缓存，提前验证参数，调用失败后伪造容错数据等，这时候就要在API中带上Stub，消费方生成Proxy实例，会把Proxy通过构造函数传给Stub，然后把Stub暴露给用户，Stub可以决定要不要去调用Proxy；



一定要写个有参构造器，我们不需要传入代理实现，dubbo会自动传入。



一般现在接口项目中写 存根接口，具体实现放在消费方或提供方都可以，只要指定即可。