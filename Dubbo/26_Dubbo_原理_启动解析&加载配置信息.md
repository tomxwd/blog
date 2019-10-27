---
title: 26_Dubbo_原理_启动解析&加载配置信息
date: 2019-09-22 16:57:12
tags: 
 - Dubbo
categories:
 - Java
 - Dubbo
---

# 26_Dubbo\_原理_启动解析&加载配置信息

Spring有个BeanDefinitionParser，Dubbo实现了这个类，叫做DubboBeanDefinitionParser；

进行debug，可以跟踪到具体解析过程。

而为什么能够解析dubbo的标签，这里是因为有个DubboNamespaceHandler类，比如application这个标签，对应解析成ApplicationConfig类。

其实大致就是解析了xml配置文件，然后把他们全部塞到具体的类中，交给Spring进行管理。

这里需要注意的是service标签用的是ServiceBean.class，reference标签用的是ReferenceBean.class；

