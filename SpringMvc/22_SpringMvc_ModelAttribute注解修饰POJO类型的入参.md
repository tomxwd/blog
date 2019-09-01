---
title: 22_SpringMvc_ModelAttribute注解修饰POJO类型的入参
date: 2019-07-25 14:44:07
tags: 
 - Spring
 - SpringMvc
categories:
 - SpringMvc
---

# 22_SpringMvc_ModelAttribute注解修饰POJO类型的入参

@ModelAttribute注解也可以用来修饰目标方法POJO类型的入参，其value属性值有以下的作用：

1. SpringMvc会使用value的属性值在implicitModel中查找对应的对象，若存在，则会直接传入到目标方法的入参中。
2. SpringMvc会以value为key，POJO类型的对象为value，存入到request中。

