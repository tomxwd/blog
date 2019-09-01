---
title: 42_MyBatis_动态sql_choose_分支选择
date: 2019-08-11 19:17:03
tags: 
 - Java
 - Mybatis
categories:
 - Mybatis
---

# 42_MyBatis\_动态sql_choose_分支选择

choose标签包括了when和otherwise，是一个分支选择的作用，类似带了break的switch-case的作用。

那么这里模拟的场景是：如果带了id，就用id来查，如果带了lastName，就用lastName来查，如果带了email，就用email查，其他情况，则查所有，并不是统一起来都用上条件；

## 接口类

```java
List<Employee> getEmpsByConditionChoose(Employee employee);
```



## sql映射文件

```xml
<select id="getEmpsByConditionChoose" resultType="top.tomxwd.mybatis.bean.Employee">
    select * from tbl_employee
    <where>
        <choose>
            <when test="id!=null">
                id=#{id}
            </when>
            <when test="lastName!=null and lastName!=''">
                last_name=#{lastName}
            </when>
            <when test="email!=null and email!=''">
                email=#{email}
            </when>
            <otherwise>
                1=1
            </otherwise>
        </choose>
    </where>
</select>
```



## 测试

```java
@Test
public void testGetEmpsByConditionChoose(){
    Employee employee = new Employee();
    employee.setId(1);
    employee.setEmail("613@qq.com");
    List<Employee> emps = empMapper.getEmpsByConditionChoose(employee);
    System.out.println("emps = " + emps);
}
```



## 结果

```
DEBUG [main] - ==>  Preparing: select * from tbl_employee WHERE id=? 
DEBUG [main] - ==> Parameters: 1(Integer)
TRACE [main] - <==    Columns: id, last_name, gender, email, d_id
TRACE [main] - <==        Row: 1, tom, 0, tom@tomxwd.top, 1
DEBUG [main] - <==      Total: 1
emps = [Employee(id=1, lastName=tom, email=tom@tomxwd.top, gender=0, dept=null)]
```

