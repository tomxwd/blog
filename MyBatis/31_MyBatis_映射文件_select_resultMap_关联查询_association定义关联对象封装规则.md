---
title: 31_MyBatis_映射文件_select_resultMap_关联查询_association定义关联对象封装规则
date: 2019-08-11 12:07:40
tags: 
 - Java
 - Mybatis
categories:
 - Mybatis
---

## 31_MyBatis\_映射文件\_select_resultMap\_关联查询_association定义关联对象封装规则

除了用级联属性来封装，我们还可以用association来定义；



## association

`<association>`也是resultMap中的一个标签，翻译过来就是联合的意思；

association里的属性：

- property：即是指定封装到哪个属性；
- javaType：属性对象对应的类型；【不可以省略】

然后再在association中定义封装规则，如id、result；



## 具体实现例子

### 接口类

```
Employee2 getEmpAndDept2(Integer id);
```



### SQL映射文件

```xml
<resultMap id="myEmpAndDpet2" type="top.tomxwd.mybatis.bean.Employee2">
    <id column="id" property="id"></id>
    <result column="last_name" property="lastName"></result>
    <result column="email" property="email"></result>
    <result column="gender" property="gender"></result>
    <association property="dept" javaType="top.tomxwd.mybatis.bean.Department">
        <id column="id" property="id"></id>
        <result column="department_name" property="departmentName"></result>
    </association>
</resultMap>

<select id="getEmpAndDept2" resultMap="myEmpAndDept">
    select emp.id id,emp.last_name lastName,emp.gender gender,emp.d_id d_id,dept.department_name departmentName
    from tbl_employee emp join tbl_department dept on emp.d_id = dept.id where emp.id=#{id}
</select>
```



### 测试

```java
@Test
public void testGetEmpAndDept2(){
    Employee2 empAndDept2 = mapper.getEmpAndDept2(1);
    System.out.println("empAndDept2 = " + empAndDept2);
}
```

