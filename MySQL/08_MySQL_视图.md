---
title: 08_MySQL_视图
date: 2020-02-17 21:48:30
tags: 
 - MySQL
categories:
 - MySQL
---

# 08_MySQL_视图

**含义：**

虚拟表，和普通表一样使用

是mysql5.1之后出现的新特性，是**通过表动态生成的数据**，视图只保存了sql逻辑，不保存查询结果；



**作用：**

- 重用sql语句
- 简化复杂的sql操作，不必知道他的查询细节
- 保护数据，提高安全性



## 创建视图

**语法：**

create view 视图名

as 查询语句；

**可以嵌套视图，即创建的视图里面使用了其他视图**；



## 查看视图

desc 视图名;

show create view 视图名;



## 使用视图

**语法：**

普通select语法，把from的表换成view视图即可；



## 修改视图

**语法：**

- 方式一：

  create or replace view 视图名 as 查询语句；

- 方式二：

  alter view 视图名 as 查询语句；



## 删除视图

**语法：**

drop view 视图名,视图名,视图名...;



## 视图的更新

**一般不允许更新操作**（增删改），如果是简单的就可以更新，下面的情况是不能更新的：

- 包含以下关键字的sql语句：
  - 分组函数
  - distinct
  - group by
  - having
  - union
  - union all
- 常量视图
- select中包含子查询
- join
- from一个不能更新的视图
- where子句的子查询引用了from子句中的表



## 视图和表的对比

**语法：**

- 视图：create view
- 表：create table



视图没有实际占用物理空间，只保存了sql逻辑，而表占有物理空间；



**使用：**

增删改查都支持，但是视图一般不能增删改；

