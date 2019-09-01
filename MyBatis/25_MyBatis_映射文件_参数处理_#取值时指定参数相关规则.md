---
title: 25_MyBatis_映射文件_参数处理_#取值时指定参数相关规则
date: 2019-08-10 20:31:42
tags: 
 - Java
 - Mybatis
categories:
 - Mybatis
---

# 25_MyBatis\_映射文件\_参数处理\_#取值时指定参数相关规则

#{}更加丰富的用法：

## 规定参数的一些规则

### 参数可以指定一个特殊的数据类型

- #{property,javaType=int,jdbcType=NUMERIC}
- #{height,javaType=double,jdbcType=NUMERIC,numericScale=2}

具体讲解：

- javaType通常可以从参数对象中来去确定
- **如果null被当作值来传递，对于所有可能为空的列，jdbcType需要被设置。**
- 对于数值类型，还可以设置小数点后保留的位数；
- mode属性允许指定IN，OUT或者INOUT参数，如果参数为OUT或INOUT，参数对象属性的真实值将会被改变，就像在获取输出参数时所期望的那样。

### 参数位置具体支持的属性

- javaType、jdbcType、mode(存储过程)、numericScale(保留几位小数)、resultMap、typeHandler(处理数据的类型处理器)、jdbcTypeName、*expression(表达式，未来准备支持的功能 )*

- 实际上通常被设置的是:

  - 可能为空的列名指定**jdbcType**

  - 在我们数据为null的时候，有些数据库可能不能识别mybatis对null的默认处理，比如Oracle（会报错）；

    报错：jdbcType OTHER:无效的类型；**因为mybatis对所有的null都映射的是JDBC OTHER类型**；对于mysql而言是可行的，但是对Oracle而言不可行，Oracle需要指定为NULL类型，即`jdbcType=NULL`

    这时候我们必须要指定jdbcType；

- #{key}：获取参数的值，预编译到sql中，安全

- ${key}：获取参数的值，拼接到sql中，有sql注入的问题。以及ORDER BY ${name}



## 小结

最好指定jdbcType，如果是Oracle的null值，需要指定为jdbcType=NULL，而mybatis默认是jdbcType=OTHER；

因为全局配置中，jdbcTypeForNull=OTHER，也可以改全局配置settings标签内。

但是mysql是两种的支持的。