---
title: 28_Dubbo_原理_服务引用流程
date: 2019-09-22 19:21:56
tags: 
 - Dubbo
categories:
 - Java
 - Dubbo
---

# 28_Dubbo\_原理_服务引用流程

![/dev-guide/images/dubbo-refer.jpg](28_Dubbo_%E5%8E%9F%E7%90%86_%E6%9C%8D%E5%8A%A1%E5%BC%95%E7%94%A8%E6%B5%81%E7%A8%8B/dubbo-refer.jpg)

1. 跟服务暴露一样，服务的引用dubbo:reference标签也有对应的类，ReferenceBean类。

2. ReferenceBean实现了Spring的FactoryBean接口，也就是工厂bean，那么在注入的时候，Spring就会调用实现了FactoryBean接口的getObject()方法来获取bean对象。
3. 首先会进行一些属性检查，然后会调用createProxy(map)方法，创建代理对象，而所有的调用信息都在map中，用于创建代理对象。
4. 创建代理对象的时候，会调用Protocol类【DubboProtocol，ReferenceProtocol】的refer(interfaceClass, urls.get(0))方法，传入接口类信息，以及url信息，这里的url信息包含了注册中心的地址信息等。
5. 同样这里是调用两个Protocol类，Dubbo的以及注册中心的。这里调用的过程跟服务提供的过程差不多，获取注册中心的信息然后doRefer()方法，在里面创建一个Directory对象，然后调用directory.subscribe()方法进行订阅服务操作；
6. 然后到DubboProtocol中，会调用getClients(url)方法进行获取客户端，然后就是创建客户端（ExchangeClient对象），然后就是跟服务提供流程差不多的，过程，建立连接到netty的底层。
7. 包装完信息之后，会返回一个Invoker对象，这个对象中就包含了所有的引用信息了。
8. 接下来就是放入到ProviderConsumerRegTable对象中，把引用的信息也保存到注册表中去。
9. 然后就创建完了代理对象，那么调用方法就是用代理对象去调用了。

