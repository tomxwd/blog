---
title: 36_MyBatis_映射文件_select_resultMap_关联查询_分步查询传递多列值&fetchType
date: 2019-08-11 15:42:00
tags: 
 - Java
 - Mybatis
categories:
 - Mybatis
---

# 36_MyBatis\_映射文件\_select_resultMap\_关联查询_分步查询传递多列值&fetchType

## 多列值传递

不管是association还是collection，在做分步查询的时候有个属性是column，那么我们要将**多个列的值传递过去**呢？

**将多列的值封装成一个map进行传递；**

column="{key1=column1,key2=column2}"

## fetchType

fetchType属性，默认是lazy，表示使用延迟加载，如果在全局配置中写了延迟加载，可以设置成eager，改成立即加载；

- lazy
- eager