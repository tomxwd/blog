---
title: 48_MyBatis_动态sql_内置参数_parameter&_databaseId
date: 2019-08-12 23:04:05
tags: 
 - Java
 - Mybatis
categories:
 - Mybatis
---

# 48_MyBatis\_动态sql\_内置参数_parameter&\_databaseId

不止是方法传递过来的参数可以被用来判断、取值。。。

mybatis默认还有两个内置参数：

- _parameter：代表整个参数
  - 单个参数：_parameter就是这个参数；
  - 多个参数：参数会被封装成一个map，那么_parameter就是代表这个map；
- _databaseId：如果配置了databaseIdProvider标签，也就是全局配置里面，那么\_databaseId就是代表当前数据库的别名；



**即**

**可以用_parameter来判断是否传了个null；**

**可以根据_databaseId来判断用什么sql语句；**

```xml
<select id="getEmpInnerParameter" resultType="top.tomxwd.mybatis.bean.Employee">
    <if test="_databaseId=='mysql'">
        select * from tbl_employee
        <if test="_parameter!=null">
            where last_name=#{lastName}
        </if>
    </if>
    <if test="_databaseId=='oracle'">
        select * from employee
        <if test="_parameter!=null">
            where last_name=#{_parameter.lastName}
        </if>
    </if>
</select>
```

