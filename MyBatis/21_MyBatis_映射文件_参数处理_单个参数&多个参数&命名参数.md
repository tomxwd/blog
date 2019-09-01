---
title: 21_MyBatis_映射文件_参数处理_POJO&Map&TO
date: 2019-08-10 10:59:25
tags: 
 - Java
 - Mybatis
categories:
 - Mybatis
---

# 21_MyBatis\_映射文件\_参数处理_POJO&Map&TO

## POJO

如果多个参数正好是我们业务逻辑的数据模型，我们就可以直接传入pojo；

- #{属性名}：取出传入的pojo的属性值；



## Map

如果多个参数没有对应的业务逻辑的数据模型（POJO），不经常使用的话，为了方便也可以传入一个Map；

- #{key}：取出map中对应的值



## TO

如果多个参数不是同一个业务模型中的数据（POJO），但是又需要经常进行使用，推荐来编写一个TO（Transfer Object）数据传输对象。

比如（伪代码）：

分页对象

Page{

​	int index;

​    int size;

}

