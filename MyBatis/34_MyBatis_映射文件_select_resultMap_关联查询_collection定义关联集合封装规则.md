---
title: 34_MyBatis_映射文件_select_resultMap_关联查询_collection定义关联集合封装规则
date: 2019-08-11 12:58:57
tags: 
 - Java
 - Mybatis
categories:
 - Mybatis
---

# 34_MyBatis\_映射文件\_select_resultMap\_关联查询_collection定义关联集合封装规则

比如一个部门下会有很多个员工，这个时候就需要封装成集合。

这时候我们就要使用resultMap中的collection标签。

## collection

定义关联集合封装的规则；

需要写几个属性

- property：在结果集中对应的属性名，如这里的emps
- ofType：这个集合里面的元素类型

再在里面定义每个元素具体的相关规则。



## 例子实现

### 接口类

```java
Department getDeptByIdPlus(Integer id);
```



### SQL映射文件

```xml
<resultMap id="myDept" type="top.tomxwd.mybatis.bean.Department">
    <id column="id" property="id"/>
    <result column="departmentName" property="departmentName"/>
    <collection property="emps" ofType="top.tomxwd.mybatis.bean.Employee2">
        <id column="eid" property="id"></id>
        <result column="last_name" property="lastName"></result>
        <result column="email" property="email"></result>
        <result column="gender" property="gender"></result>
    </collection>
</resultMap>
<select id="getDeptByIdPlus" resultMap="myDept">
    select d.id,d.department_name departmentName,e.id eid,e.last_name last_name,e.email email,e.gender gender
    from tbl_employee e
    join tbl_department d on e.d_id=d.id
    where d.id = #{id}
</select>
```



### 测试

```java
@Test
public void testGetDeptPlus(){
    Department deptByIdPlus = this.deptMapper.getDeptByIdPlus(1);
    System.out.println("deptByIdPlus = " + deptByIdPlus);
}
```



### 输出结果

```
DEBUG [main] - ==>  Preparing: select d.id,d.department_name departmentName,e.id eid,e.last_name last_name,e.email email,e.gender gender from tbl_employee e join tbl_department d on e.d_id=d.id where d.id = ? 
DEBUG [main] - ==> Parameters: 1(Integer)
TRACE [main] - <==    Columns: id, departmentName, eid, last_name, email, gender
TRACE [main] - <==        Row: 1, 部门1, 1, tom, tom@tomxwd.top, 0
TRACE [main] - <==        Row: 1, 部门1, 4, ttt, 487@qq.com, 0
TRACE [main] - <==        Row: 1, 部门1, 5, tom, 613@qq.com, 1
DEBUG [main] - <==      Total: 3
deptByIdPlus = Department(id=1, departmentName=部门1, emps=[Employee2(id=1, lastName=tom, email=tom@tomxwd.top, gender=0, dept=null), Employee2(id=4, lastName=ttt, email=487@qq.com, gender=0, dept=null), Employee2(id=5, lastName=tom, email=613@qq.com, gender=1, dept=null)])
```

可以看到结果输出正确，collection也是嵌套结果集的方式，定义集合类型元素的封装规则。