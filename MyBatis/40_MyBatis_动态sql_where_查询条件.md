---
title: 40_MyBatis_动态sql_where_查询条件
date: 2019-08-11 18:46:05
tags: 
 - Java
 - Mybatis
categories:
 - Mybatis
---

# 40_MyBatis\_动态sql_where_查询条件

在39节中，我们很明显可以发现，如果说不带上id属性的话，会多了个and，因为其他条件都有and；

这里一般有两种解决方案，下面一一介绍；



## 方案一（where后面加1=1）

```xml
<select id="getEmpsByConditionIf" resultType="top.tomxwd.mybatis.bean.Employee">
    select * from tbl_employee
    WHERE 1=1
    <if test="id!=null">
        and id=#{id}
    </if>
    <if test="lastName!=null and lastName!=''">
        and last_name like "%"#{lastName}"%"
    </if>
    <if test="email!=null and email.trim()!=''">
        and email=#{email}
    </if>
    <if test="gender==0 or gender==1">
        and gender=#{gender}
    </if>
</select>
```

除了在where后面加上1=1，还要在判断id的前面加and；



## 方案二（使用where标签）

```xml
<select id="getEmpsByConditionIf" resultType="top.tomxwd.mybatis.bean.Employee">
    select * from tbl_employee
    <where>
        <if test="id!=null">
            and id=#{id}
        </if>
        <if test="lastName!=null and lastName!=''">
            and last_name like "%"#{lastName}"%"
        </if>
        <if test="email!=null and email.trim()!=''">
            and email=#{email}
        </if>
        <if test="gender==0 or gender==1">
            and gender=#{gender}
        </if>
    </where>
</select>
```

mybatis的where标签甚至还可以根据情况判断是否加and；



### 测试

```java
@Test
public void testIf(){
    Employee emp = new Employee();
    emp.setLastName("t");
    emp.setEmail("613@qq.com");
    List<Employee> emps = empMapper.getEmpsByConditionIf(emp);
    System.out.println("emps = " + emps);
}
```



### 结果

```
DEBUG [main] - ==>  Preparing: select * from tbl_employee WHERE last_name like "%"?"%" and email=? 
DEBUG [main] - ==> Parameters: t(String), 613@qq.com(String)
TRACE [main] - <==    Columns: id, last_name, gender, email, d_id
TRACE [main] - <==        Row: 5, tom, 1, 613@qq.com, 1
DEBUG [main] - <==      Total: 1
emps = [Employee(id=5, lastName=tom, email=613@qq.com, gender=1, dept=null)]
```

可以根据结果看到，lastName之前的and被去掉了。

而且注意**在where标签的情况下！and一定是要在前面拼的，不可以放在后面**！

如果and想放在后面，可以用`<trim>`标签;