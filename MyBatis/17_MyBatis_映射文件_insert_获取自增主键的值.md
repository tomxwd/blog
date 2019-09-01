---
title: 17_MyBatis_映射文件_insert_获取自增主键的值
date: 2019-08-09 23:36:15
tags: 
 - Java
 - Mybatis
categories:
 - Mybatis
---

# 17_MyBatis\_映射文件\_insert_获取自增主键的值

在原生JDBC中，我们可以用结果集的getGeneratedKeys()来返回自增主键，那么在mybatis中如何取得？

mybatis也是利用statement.getGeneratedKeys()方法的。

在映射文件的标签上有个**useGeneratedKyes="ture"**使用自增主键获取主键策略，写上后，还要加上**keyProperty="id"**来指定对应的主键属性，也就是mybatis获取到这个主键值之后封装给bean的哪个属性。



## 映射文件配置

```xml
<insert id="addEmp" parameterType="top.tomxwd.mybatis.bean.Employee" useGeneratedKeys="true" keyProperty="id">
    insert into tbl_employee (last_name,email,gender) values (#{lastName},#{email},#{gender})
</insert>
```



## 测试

```java
@Test
public void testAddEmp(){
    Employee emp = new Employee(null, "tom", "613@qq.com", "1");
    Integer i = mapper.addEmp(emp);
    System.out.println("i = " + i);
    System.out.println("emp = " + emp);
}
```

