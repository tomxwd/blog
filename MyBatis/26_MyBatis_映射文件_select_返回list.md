---
title: 26_MyBatis_映射文件_select_返回list
date: 2019-08-10 22:03:37
tags: 
 - Java
 - Mybatis
categories:
 - Mybatis
---

# 26_MyBatis\_映射文件\_select_返回list

select元素

- Select元素用来定义查询操作。
- id：唯一标识符
  - 用来引用这条语句，需要和接口的方法名一致。
- parameterType：参数类型
  - 可以不传，MyBatis会根据TypeHandler自动推断
- resultType：返回值类型
  - 别名或者全类名，如果返回的是集合，定义集合中的元素类型，不能和resultMap同时使用。



## 接口类

```java
List<Employee> getEmps();
```



## SQL映射文件

```xml
    <select id="getEmps" resultType="Employee">
        select * from tbl_employee;
    </select>
```



## 测试

```java
@Test
public void testGetEmps(){
    List<Employee> emps = mapper.getEmps();
    System.out.println("emps = " + emps);
}
```

