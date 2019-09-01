---
title: 24_MyBatis_映射文件_参数处理_#与$取值区别
date: 2019-08-10 19:58:57
tags: 
 - Java
 - Mybatis
categories:
 - Mybatis
---

# 24_MyBatis\_映射文件\_参数处理_#与$取值区别

我们知道用#{}可以取得map或者pojo对象属性的值；

对于mybatis而言，还可以用${}来取值。

## 验证一下

映射文件：

```xml
<select id="getEmpByIdAndLastName" resultType="Employee">
    select * from tbl_employee where id=${id} and last_name=#{lastName}
</select>
```

拿到运行的sql进行分析：

```
select * from tbl_employee where id=1 and last_name=?
```

可以发现，跟之前不一样，sql语句里面用${id}，在sql上是直接把值放到了语句中的。

也就是说：

- #{}：是以预编译的形式，将参数设置到sql语句中PerparedStatement（**可以防止sql注入**）
- ${}：取出的值直接拼装在sql语句中（**会有安全问题**）
- 还有就是用#{}，最终传入值的时候会有加单引号，而${}则没有；

大多数情况下，我们取参数的值我们都应该使用#{}；

对于原生jdbc不支持占位符的地方我们就可以使用${}来取值，比如**表名**，**order by**之类的。

**比如分表：按照年份分表拆分**

select * from 2016_salary where XXX order by name，order；

可以使用如下形式：

select * from ${year}_salary where XXX order by ${f_name}，${order} ;