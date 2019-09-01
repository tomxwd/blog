---
title: 41_MyBatis_动态sql_trim_自定义字符串截取
date: 2019-08-11 18:58:25
tags: 
 - Java
 - Mybatis
categories:
 - Mybatis
---

# 41_MyBatis\_动态sql_trim_自定义字符串截取

如果说你把and拼在了条件后面，那么用where标签是无法解决的，那么我们可以用trim标签来解决，自定义字符串的截取规则；



## trim相关的属性

- prefix：前缀，trim标签里面是整个字符串拼好之后的结果，而prefix是给拼好的字符串加一个前缀，比如可以写where；
- prefixOverrides：前缀覆盖，去掉整个字符串前面多余的字符，这里我们模拟的是and在后面，所以没有多余的。
- suffix：后缀，整体加后缀，跟prefix差不多功能；
- suffixOverrides：后缀覆盖，跟prefixOverrie类似，去掉后面多余的，这里我们使用这个去除后面多的and；



## 相关代码

### 接口类

```java
List<Employee> getEmpsByConditionTrim(Employee employee);
```





### 映射文件

```xml
<select id="getEmpsByConditionTrim" resultType="top.tomxwd.mybatis.bean.Employee">
    select * from tbl_employee
    <trim prefix="where" prefixOverrides="" suffix="" suffixOverrides="and">
        <if test="id!=null">
            id=#{id} and
        </if>
        <if test="lastName!=null and lastName!=''">
            last_name like "%"#{lastName}"%" and
        </if>
        <if test="email!=null and email.trim()!=''">
            email=#{email} and
        </if>
        <if test="gender==0 or gender==1">
            gender=#{gender}
        </if>
    </trim>
</select>
```



### 测试

```java
@Test
public void testGetEmpsByConditionTrim(){
    Employee emp = new Employee();
    emp.setLastName("t");
    emp.setEmail("613@qq.com");
    List<Employee> emps = empMapper.getEmpsByConditionTrim(emp);
    System.out.println("emps = " + emps);
}
```



### 结果

```
DEBUG [main] - ==>  Preparing: select * from tbl_employee where last_name like "%"?"%" and email=? 
DEBUG [main] - ==> Parameters: t(String), 613@qq.com(String)
TRACE [main] - <==    Columns: id, last_name, gender, email, d_id
TRACE [main] - <==        Row: 5, tom, 1, 613@qq.com, 1
DEBUG [main] - <==      Total: 1
emps = [Employee(id=5, lastName=tom, email=613@qq.com, gender=1, dept=null)]
```

