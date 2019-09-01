---
title: 11_JDBC_使用BeanUtils工具类操作JavaBean
date: 2019-08-22 16:41:41
tags: 
 - Java
categories:
 - JDBC
---

# 11_JDBC_使用BeanUtils工具类操作JavaBean

Java类的属性：

1. 在JavaEE中，java类的属性通过getter、setter来定义：get（或set）方法，去除get（或set）后，首字母小写即为Java类的属性。
2. 而以前叫的属性，即成员变量，称之为字段。
3. 操作java类的属性有一个工具包：beanutils
   - 搭建环境：
     - commons-beanutils.jar
     - commons-logging.jar
   - 常用方法
     - BeanUtils.setProperty()
     - BeanUtils.getProperty()
4. 一般情况下，字段名和属性名都一致

