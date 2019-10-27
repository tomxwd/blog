---
title: 15_Dubbo_配置_超时&配置覆盖关系
date: 2019-09-22 09:53:45
tags: 
 - Dubbo
categories:
 - Java
 - Dubbo
---

# 15_Dubbo\_配置_超时&配置覆盖关系



## 超时属性设置

在dubbo:reference标签中还有一个timeout属性，是超时设置，单位为毫秒，默认是1000毫秒，也就是1秒，如果服务提供方需要2秒才能执行完程序，会抛出超时异常。

dubbo:reference是单个服务设置的，而dubbo:consumer则是全局配置的。



## 配置覆盖关系

- 方法级优先，接口级次之，全局配置再次之。
- 如果级别一样，则消费方优先，提供方次之。

![dubbo-config-override](15_Dubbo_%E9%85%8D%E7%BD%AE_%E8%B6%85%E6%97%B6&%E9%85%8D%E7%BD%AE%E8%A6%86%E7%9B%96%E5%85%B3%E7%B3%BB/dubbo-config-override.jpg)

建议由服务提供方设置超时，因为一个方法需要执行多长时间，服务提供方更清楚，如果一个消费方同时引用多个服务，就不需要关心每个服务的超时设置；

