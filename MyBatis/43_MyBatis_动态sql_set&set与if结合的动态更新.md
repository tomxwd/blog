---
title: 43_MyBatis_动态sql_set&set与if结合的动态更新
date: 2019-08-11 19:28:48
tags: 
 - Java
 - Mybatis
categories:
 - Mybatis
---

# 43_MyBatis_动态sql_set&set与if结合的动态更新

如果我们用普通的set，然后结合if标签，会发现如果条件不足，会报错，因为结尾的逗号（，）会生效，这时候我们可以用mybatis的set标签来解决，会根据情况判断是否删除后面的逗号，达到动态sql的目的；

**当然我们可以用trim标签来解决，这里不演示**;

```xml
<trim prefix="set" suffixOverrides=",">
```



## 接口类

```java
Integer updateEmp(Employee employee);
```



## sql映射文件

```xml
<update id="updateEmp">
    update tbl_employee
    <set>
        <if test="lastName!=null and lastName!=''">
            last_name=#{lastName},
        </if>
        <if test="email!=null and email!=''">
            email=#{email},
        </if>
        <if test="gender!=null and gender!=''">
            gender=#{gender}
        </if>
    </set>
    where id=#{id}
</update>
```



## 测试

```java
@Test
public void testUpdateEmp(){
    Employee emp = new Employee();
    emp.setId(1);
    emp.setLastName("tom");
    emp.setEmail("tom@qq.com");
    Integer integer = empMapper.updateEmp(emp);
    System.out.println("integer = " + integer);
}
```



## 输出

```
DEBUG [main] - ==>  Preparing: update tbl_employee SET last_name=?, email=? where id=? 
DEBUG [main] - ==> Parameters: tom(String), tom@qq.com(String), 1(Integer)
DEBUG [main] - <==    Updates: 1
integer = 1
```

