---
title: 27_MyBatis_映射文件_select_记录封装map
date: 2019-08-10 22:28:43
tags: 
 - Java
 - Mybatis
categories:
 - Mybatis
---

# 27_MyBatis\_映射文件\_select_记录封装map

结果集返回map，key就是列名，值就是对应的值；

## 查询单个对象的时候这样使用

### 接口类

```java
Map<String,Object> getEmpMapById(Integer id);
```

### sql映射文件

```xml
<select id="getEmpMapById" resultType="map">
    select * from tbl_employee where id=#{id}
</select>
```

### 测试

```java
@Test
public void testGetEmpMapById(){
    Map<String, Object> empMapById = mapper.getEmpMapById(1);
    System.out.println("empMapById = " + empMapById);
}
```

## 查询多个对象的时候，类似list

`Map<Integer,Employee>`key是主键，而值是记录封装后的javabean；

两个需求，一个是key，一个是value

- key用注解**@MapKey**来指定，比如@MapKey("id")就指key用id来封装；
- value则是在返回值类型那里设定，也就是resultType，指定封装的javabean等即可。

### 接口类

MapKey的值是用于告诉mybatis封装这个map的时候用哪个属性作为map的key

```java
@MapKey("id")
Map<Integer,Employee> getEmpsMap();
```

### sql映射文件

```xml
<select id="getEmpsMap" resultType="Employee">
    select * from tbl_employee;
</select>
```

### 测试

```java
@Test
public void testGetEmpsMap(){
    Map<Integer, Employee> empsMap = mapper.getEmpsMap();
    System.out.println("empsMap = " + empsMap);
}
```

