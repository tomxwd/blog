---
title: 07_tomcat_分页
date: 2019-11-24 17:46:39
tags: 
 - tomcat
categories:
 - tomcat
---

# 07_tomcat_分页

**分页原理：limit**

select * from table limit 起始索引,要取出的个数







## 后端部分

类信息：

- 当前页：currentPage
- 总页：totalPage
- 总记录数：totalCount
- 每页条数：pageSize
- 数据库用的起始索引：index
- 传输对象：pageData



## 前端部分

首页--上一页--页码--当前页--页码---末页----共n页，每页m条记录----跳转到第 x 页----确定按钮

显示信息：

- 当前页：用户传入
- 总页：根据总记录数计算出来
- 总记录数：后端传入
- 每页条数：前端控制，用户可改
- 传输对象：后端传入

