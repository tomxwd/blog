---
title: 15_MyBatis_小结(2)
date: 2019-08-09 19:25:36
tags: 
 - Java
 - Mybatis
categories:
 - Mybatis
---

# 15_MyBatis_小结(2)

>[官方文档](http://www.mybatis.org/mybatis-3/zh/)

全局配置文件中，还有一个标签是`<objectFactory>`标签，对象工厂，是mybatis查出对象要封装数据的时候使用的对象工厂，一般不会进行改变，所以不需要进行设置。更多的可以看官方文档使用即可。

还有注意标签的前后顺序，需要按顺序写，不过一般也会报错显示出来，看报错信息即可。