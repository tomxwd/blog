---
title: 25_Shiro_缓存
date: 2019-08-30 16:15:32
tags: 
 - Java
 - Shiro
categories:
 - Shiro
---

# 25_Shiro_缓存

## CacheManagerAware接口

Shiro内部相应的组件（DefaultSecurityManager）会自动检测相应的对象（如Realm）**是否实现了CacheManagerAware并自动注入相应的CacheManager**；



## Realm缓存

- Shiro提供了CachingRealm，其实现了CacheManagerAware接口，提供了缓存的一些基础实现；
- **AuthenticatingRealm及AuthorizingRealm**也分别提供了对AuthenticationInfo和AuthorizationInfo信息的缓存；



## Session缓存

- 如SecurityManager实现了SessionSecurityManager，其会判断SessionManager是否实现了CacheManagerAware接口，如果实现了会把CacheManager设置给它；
- SessionManager也会判断相应的SessionDAO（如继承自CachingSessionDAO）是否实现了CacheManagerAware，如果实现了会把CacheManager设置给它；
- 设置了缓存的SessionManager，查询时会先查缓存，如果找不到才找数据库。



