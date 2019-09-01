---
title: 35_MyBatis_映射文件_select_resultMap_关联查询_collection分步查询&延迟加载
date: 2019-08-11 15:25:47
tags: 
 - Java
 - Mybatis
categories:
 - Mybatis
---

# 35_MyBatis\_映射文件\_select_resultMap\_关联查询_collection分步查询&延迟加载
要用分步查询，就需要两边同时定义方法，然后相互调用。



## 全局配置

```xml
<settings>
    <!--开启驼峰命名-->
    <setting name="mapUnderscoreToCamelCase" value="true"/>
    <setting name="lazyLoadingEnabled" value="true"/>
    <setting name="aggressiveLazyLoading" value="false"/>
</settings>
```



## 员工接口

```java
List<Employee2> getEmpByDid(Integer id);
```



## 员工SQL映射文件

```xml
<select id="getEmpByDid" resultType="top.tomxwd.mybatis.bean.Employee2">
    select * from tbl_employee where d_id=#{id}
</select>
```



## 部门接口

```java
Department getDeptById2(Integer id);
```



## 部门SQL映射文件

```xml
<resultMap id="myDept2" type="top.tomxwd.mybatis.bean.Department">
    <id column="id" property="id"></id>
    <result column="department_name" property="departmentName"></result>
    <collection property="emps" select="top.tomxwd.mybatis.mapper.EmployeeMapperPlus.getEmpByDid" column="id"></collection>
</resultMap>
<select id="getDeptById2" resultMap="myDept2">
    select * from tbl_department where id=#{id}
</select>
```



## 测试

```java
@Test
public void testGetDept2(){
    Department deptById2 = this.deptMapper.getDeptById2(1);
    System.out.println(deptById2.getDepartmentName());
    System.out.println("-----------------------------------");
    System.out.println(deptById2.getEmps());
}
```



## 输出结果

```
DEBUG [main] - ==>  Preparing: select * from tbl_department where id=? 
DEBUG [main] - ==> Parameters: 1(Integer)
TRACE [main] - <==    Columns: id, department_name
TRACE [main] - <==        Row: 1, 部门1
DEBUG [main] - <==      Total: 1
部门1
-----------------------------------
DEBUG [main] - ==>  Preparing: select * from tbl_employee where d_id=? 
DEBUG [main] - ==> Parameters: 1(Integer)
TRACE [main] - <==    Columns: id, last_name, gender, email, d_id
TRACE [main] - <==        Row: 1, tom, 0, tom@tomxwd.top, 1
TRACE [main] - <==        Row: 4, ttt, 0, 487@qq.com, 1
TRACE [main] - <==        Row: 5, tom, 1, 613@qq.com, 1
DEBUG [main] - <==      Total: 3
[Employee2(id=1, lastName=tom, email=tom@tomxwd.top, gender=0, dept=null), Employee2(id=4, lastName=ttt, email=487@qq.com, gender=0, dept=null), Employee2(id=5, lastName=tom, email=613@qq.com, gender=1, dept=null)]
```

可以看到是延迟加载的；

