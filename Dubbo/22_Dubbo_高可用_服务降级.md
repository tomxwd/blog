---
title: 22_Dubbo_高可用_服务降级
date: 2019-09-22 12:40:48
tags: 
 - Dubbo
categories:
 - Java
 - Dubbo
---

# 22_Dubbo\_高可用_服务降级

当服务器压力剧增的情况下，根据实际业务情况以及流量，对一些服务和页面有策略的不处理或换种简单的方式处理，从而释放服务器资源以保证核心交易正常运作或高效运作。



可以通过服务降级功能临时屏蔽某个出错的非关键服务，并定义降级后的返回策略。

向注册中心写入动态配置覆盖规则：

```java
RegistryFactory registryFactory = ExtensionLoader.getExtensionLoader(RegistryFactory.class).getAdaptiveExtension();
Registry registry = registryFactory.getRegistry(URL.valueOf("zookeeper://127.0.0.1:2181"));
registry.register(URL.valueOf("override://0.0.0.0/com.foo.BarService?category=configurators&dynamic=false&application=foo&mock=force:return+null"));
```



其中：

两种方式：

- mock=force:return+null：表示消费方对该服务的方法调用都直接返回null值，不发起远程调用。用来屏蔽不重要服务不可用时对调用方的影响。
- 还可以改为mock=fail:return:null：表示消费方对该服务的方法调用在失败后，再返回null值，不抛异常。用来容忍不重要服务不稳定时对调用方的影响。



可以在控制台直接进行配置：

屏蔽：不调用远程服务，直接返回为null

容错：调用失败的时候，才返回为null



