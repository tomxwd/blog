---
title: 01_Shiro_简介
date: 2019-08-28 13:59:59
tags: 
 - Java
 - Shiro
categories:
 - Shiro
---

# 01_Shiro_简介

## Shiro简介

- Apache Shiro是Java的一个安全（权限）框架。
- Shiro可以非常容易的开发出足够好的应用，其不仅可以用在JavaSE环境，也可以用在JavaEE环境。
- Shiro可以完成：认证、授权、加密、会话管理、与Web集成、缓存等。
- 下载：[Shiro官网](http://shiro.apache.org/)



## Shiro功能简介

![功能简介](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/Shiro/01%E5%8A%9F%E8%83%BD%E7%AE%80%E4%BB%8B.png)

| 基本功能点      | 简要介绍                                   | 介绍                                                         |
| --------------- | ------------------------------------------ | ------------------------------------------------------------ |
| Authentication  | 身份认证/登录                              | 验证用户是不是拥有相应的身份                                 |
| Authorization   | 授权，即权限验证                           | 验证某个已经认证的用户是否拥有某个权限；即判断用户是否能进行什么操作，比如：验证某个用户是否拥有某个角色。或者细粒度的验证某个用户对某个资源是否具有某个权限； |
| Session Manager | 会话管理                                   | 即用户登录后就是一次会话，在没有退出之前，它的所有信息都在会话中；**会话可以是普通JavaSE环境，也可以是Web环境的**； |
| Cryptography    | 加密                                       | 保护数据的安全性，如密码加密存储到数据库，而不是明文存储；   |
| Web Support     | Web支持                                    | 可以非常容易的集成到Web环境；                                |
| Caching         | 缓存                                       | 比如用户登录后，其用户信息、拥有的角色、权限不必每次都去查，这样可以提高效率； |
| Concurrency     | 多线程应用的并发验证                       | 即如在一个线程中开启另一个线程，能把权限自动传播过去。       |
| Testing         | 测试                                       | 提供测试支持。                                               |
| Run As          | 允许一个用户假装为另一个用户的身份进行访问 | 允许一个用户假装为另一个用户（如果他们允许）的身份进行访问； |
| Remember Me     | 记住我                                     | 这是个非常常见的功能，即一次登录之后，下次再来的话不用登录了。 |



## Shiro架构（Shiro外部来看）

![Shiro架构(外部)](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/Shiro/01Shiro%E6%9E%B6%E6%9E%84(%E5%A4%96%E9%83%A8).png)

| 点              | 介绍                                                         |
| --------------- | ------------------------------------------------------------ |
| Subject         | **应用代码直接交互的对象是Subject**，也就是说Shiro的对外API核心就是Subject。**Subject代表了当前的“用户”**，这个用户不一定是一个具体的人，与当前应用交互的任何东西都是Subject，如网络爬虫，机器人等；**与Subject的所有交互都会委托给SecurityManager;Subject其实是一个门面，SecurityManager才是实际的执行者**； |
| SecurityManager | 安全管理器；即**所有与安全有关的操作都会与SecurityManager交互**；且其管理着所有Subject；可以看出它是**Shiro的核心**，它**负责与Shiro的其他组件进行交互**，它相当于SpringMvc中的DispatcherServlet的角色； |
| Realm           | Shiro**从Realm获取安全数据（如用户、角色、权限）**，就是说SecurityManager要验证用户身份，那么它需要从Realm获取相应的用户进行比较以确定用户身份是否合法；也需要从Realm得到用户相应的角色/权限进行验证用户是否能进行操作；可以把Realm看成DataSource； |



## Shiro架构（Shiro内部来看）

![Shiro架构(内部)](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/Shiro/01Shiro%E6%9E%B6%E6%9E%84(%E5%86%85%E9%83%A8).png)

| 点              | 介绍                                                         |
| --------------- | ------------------------------------------------------------ |
| Subject         | 任何可以与应用交互的“用户”                                   |
| SecurityManager | 相当于SpringMvc中的DispatherServlet；是Shiro的心脏；所有具体的交互都会通过SecurityManager进行控制；它管理着所有Subject、且负责进行认证、授权、会话以及缓存的管理。 |
| Authenticator   | **负责Subject认证**，是一个扩展点，可以自定义实现；可以使用认证策略（Authentication Strategy），即什么情况下算用户认证通过了。 |
| Authorizer      | **授权器**，即访问控制器，用来决定主体是否有权限进行相应的操作；即控**制着用户能访问应用中的那些功能；** |
| Realm           | 可以有一个或多个Realm，可以认为是安全实体数据源，即用于获取安全试题的；可以是JDBC实现，也可以是内存实现等等；由用户提供，所以一般在应用中都需要实现自己的Realm； |
| SessionManager  | **管理Session生命周期的组件**；而Shiro并不仅仅可以用在Web环境，也可以用在普通的JavaSE环境； |
| CacheManager    | **缓存控制器**，用来管理如用户、角色、权限等的缓存的；因为这些数据基本上很少改变、放到缓存中后可以提高访问的性能； |
| Cryptography    | **密码模块**，Shiro提高了一些常见的加密组件用于如密码加密/解密等。 |

