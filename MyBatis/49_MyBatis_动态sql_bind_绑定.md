---
title: 49_MyBatis_动态sql_bind_绑定
date: 2019-08-12 23:09:43
tags: 
 - Java
 - Mybatis
categories:
 - Mybatis
---

# 49_MyBatis\_动态sql_bind_绑定

bind：可以将OGNL表达式的值绑定到一个变量中，方便后面引用这个变量的值。

```xml
<select id="getEmpInnerParameter" resultType="top.tomxwd.mybatis.bean.Employee">
    <bind name="_lastName" value="'%'+lastName+''"/>
    <if test="_databaseId=='mysql'">
        select * from tbl_employee
        <if test="_parameter!=null">
            where last_name like #{_lastName}
        </if>
    </if>
</select>
```

即看第二行，以及第六行。

**bind的value为"'%'+lastName+'%'"这种形式，可以进行模糊查询**;