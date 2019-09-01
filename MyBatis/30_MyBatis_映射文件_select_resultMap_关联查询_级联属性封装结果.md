---
title: 30_MyBatis_映射文件_select_resultMap_关联查询_级联属性封装结果
date: 2019-08-11 11:06:31
tags: 
 - Java
 - Mybatis
categories:
 - Mybatis
---

# 30_MyBatis\_映射文件\_select_resultMap\_关联查询_级联属性封装结果

级联属性：即用.的形式来做，比如1:1的关系；

一个员工对应一个部门，具体实现代码见下；



## 接口类

```java
Employee2 getEmpAndDept(Integer id);
```



## SQL映射文件

```xml
<resultMap id="myEmpAndDept" type="top.tomxwd.mybatis.bean.Employee2">
    <id column="id" property="id"></id>
    <result column="last_name" property="lastName"></result>
    <result column="email" property="email"></result>
    <result column="gender" property="gender"></result>
    <result column="d_id" property="dept.id"></result>
    <result column="departmentName" property="dept.departmentName"></result>
</resultMap>

<select id="getEmpAndDept" resultMap="myEmpAndDept">
    select emp.id id,emp.last_name lastName,emp.gender gender,emp.d_id d_id,dept.department_name departmentName
    from tbl_employee emp join tbl_department dept on emp.d_id = dept.id where emp.id=#{id}
</select>
```



## 测试

```java
@Test
public void testGetEmpAndDept(){
    Employee2 empAndDept = mapper.getEmpAndDept(1);
    System.out.println("empAndDept = " + empAndDept);
}
```



## 总结

联合查询：使用级联属性封装结果集，即属性.属性即可赋值，重要的是要把sql的字段信息准确对应到javabean的信息。