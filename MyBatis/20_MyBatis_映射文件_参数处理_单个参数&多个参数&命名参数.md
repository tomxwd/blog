---
title: 20_MyBatis_映射文件_参数处理_单个参数&多个参数&命名参数
date: 2019-08-10 09:50:44
tags: 
 - Java
 - Mybatis
categories:
 - Mybatis
---

# 20_MyBatis\_映射文件\_参数处理_单个参数&多个参数&命名参数

参数处理规则：#{参数名}：取出参数



## 单个参数

如果是单个参数，mybatis不会做特殊处理，也就是说，**单个参数的情况下，就算你随便输入参数名都可以生效**;



## 多个参数

```
org.apache.ibatis.binding.
BindingException: 
Parameter 'id' not found. 
Available parameters are [0, 1, param1, param2]
```

多个参数的情况mybatis会做特殊处理，会把多个参数封装成一个map来处理，默认会把key：param1到paramN，value：传入的参数值，#{}就是从map中按照key来取值，或者用索引来取值。

如果直接用#{参数名}来取，运行的时候会直接报错，显示只有参数0，1；

**解决方法：**

- 在取值的时候用#{0}#{1}或者#{param1}#{param2}来取值。

### 接口文件

```java
Employee getEmpByIdAndLastName(Integer id,String lastName);
```

### 映射文件配置

```xml
<select id="getEmpByIdAndLastName" resultType="Employee">
    select * from tbl_employee where id=#{param1} and last_name=#{param2}
</select>
```



## 命名参数

明确指定封装参数时map的key；

用`@Param`注解来指定，比如`@Param("id")`，多个参数会被封装成一个map，key这时候用的是`@Param`内指定的值。

这时候就可以直接用#{参数名}来取值了。

### 接口文件

```java
Employee getEmpByIdAndLastName(@Param("id") Integer id,@Param("lastName") String lastName);
```

### 映射文件配置

```xml
<select id="getEmpByIdAndLastName" resultType="Employee">
    select * from tbl_employee where id=#{id} and last_name=#{lastName}
</select>
```