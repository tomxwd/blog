---
title: 19_Dubbo_配置_与SpringBoot整合的三种方式
date: 2019-09-22 10:53:16
tags: 
 - Dubbo
categories:
 - Java
 - Dubbo
---

# 19_Dubbo\_配置_与SpringBoot整合的三种方式



## 第一种

1. 导入dubbo-starter，在application.properties中配置属性；
2. 使用@Service【暴露服务】，使用@Reference【引用服务】；
3. 要用@EnableDubbo开启基于注解的dubbo功能【主要是用于包扫描】



## 第二种

保留dubbo配置文件的方式

即provider.xml以及consumer.xml配置文件，这时候我们需要吧@EnableDubbo注解去掉，使用@ImportResource(locations="classpath:provider.xml")注解引入dubbo配置文件接口

也要导入dubbo-starter；



## 第三种

使用注解API的方式

1. 每个dubbo.xml配置中的dubbo标签都有对应的Config组件，如dubbo:application对应的是ApplicationConfig类，手动创建好各个组件，注入到Spring容器中即可。
2. 启动类中加@DubboComponentScan，这个注解也可以用@EnableDubbo代替，指定scanBasePackages=""属性
3. 还是用@Service暴露服务，@Reference引用服务。

