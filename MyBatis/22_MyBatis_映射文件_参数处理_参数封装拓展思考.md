---
title: 22_MyBatis_映射文件_参数处理_参数封装拓展思考
date: 2019-08-10 11:08:31
tags: 
 - Java
 - Mybatis
categories:
 - Mybatis
---

# 22_MyBatis\_映射文件\_参数处理_参数封装拓展思考

## 两个参数，一个用了@Param注解，一个没用，如何解决？

```java
Employee getEmpByIdAndLastName(@Param("id") Integer id, String lastName);
```

- id===>#{id/param1}
- lastName===>#{param2}



## 两个参数，一个是基本数据类型，一个是pojo，怎么取值？

```java
Employee getEmpByIdAndLastName(Integer id,@Param("e") Employee emp);
```

- id===>#{param1}
- lastName===>#{param2.lastName/e.lastName}



## 一个参数，但是是List类型的，需要取出其中的一个值

** 特别注意：如果是Collection（List、Set）类型，或者是数组，也会特殊处理，也是把传入的Collection封装在map中。**

key使用的是：

- Collection（collection）
- List（list）
- 数组（array）

```java
public Employee getEmpById(List<Integer> ids);
```

取出第一个id的值：

- 取出第一个id的值：#{list[0]}，#{collection[0]}