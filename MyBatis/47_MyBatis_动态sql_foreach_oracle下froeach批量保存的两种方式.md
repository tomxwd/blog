---
title: 47_MyBatis_动态sql_foreach_oracle下froeach批量保存的两种方式
date: 2019-08-12 22:39:30
tags: 
 - Java
 - Mybatis
categories:
 - Mybatis
---

# 47_MyBatis_动态sql_foreach_oracle下froeach批量保存的两种方式



## 第一种方式

### sql映射文件

```xml
<insert id="insertEmpsWithOracle">
    <foreach open="begin" collection="emps" item="emp" close="end">
        insert into xxx (Xxx,Xxx...) values(Xxx,Xxx...);
    </foreach>
</insert>
```

**记得end后面要有分号**



## 第二种方式

### sql映射文件

```xml
<insert id="insertEmpsWithOracle">
    insert into employee(employy_id,last_name,email)
    <foreach open="select employee_seq.nextval,last_name,email from (" collection="emps" item="emp" separator="union" close=")">
        select #{emp.lastName} last_name,#{emp.email} email from dual
    </foreach>
</insert>
```

