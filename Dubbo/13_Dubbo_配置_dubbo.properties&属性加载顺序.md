---
title: 13_Dubbo_配置_dubbo.properties&属性加载顺序
date: 2019-09-21 22:40:05
tags: 
 - Dubbo
categories:
 - Java
 - Dubbo
---

# 13_Dubbo\_配置_dubbo.properties&属性加载顺序

![dubbo-properties-override.jpg](13_Dubbo_%E9%85%8D%E7%BD%AE_dubbo.properties&%E5%B1%9E%E6%80%A7%E5%8A%A0%E8%BD%BD%E9%A1%BA%E5%BA%8F/dubbo-properties-override.jpg)

这里的dubbo.xml也可以说是配置文件，在spring中的xml配置文件，在SpringBoot中的application.properties文件。

这里演示一下，application.properties和dubbo.properties的优先级：

1. 在启动的时候加虚拟机参数：

   指定端口为20880

   ```
   -Ddubbo.protocol.port=20880
   ```

2. 在application.properties中：

   指定端口为20881

   ```properties
   dubbo.protocol.port=20881
   ```

3. 在dubbo.properties中：

   指定端口为20882

   ```properties
   dubbo.protocol.port=20882
   ```

最后启动项目到管理控制台看看是哪个端口生效，从控制台输出也可以看到结果；

20880>20881>20882

即虚拟机参数>application.properties>dubbo.properties