---
title: 50_MyBatis_动态sql_sql_抽取可重用的sql片段
date: 2019-08-12 23:22:39
tags: 
 - Java
 - Mybatis
categories:
 - Mybatis
---

# 50_MyBatis\_动态sql_sql_抽取可重用的sql片段

## sql标签与include标签基本使用

sql标签，它的作用就是拿来抽取可重用的sql片段，方便后面进行引用，比如我们的select xxx,xxx需要写很长一段，而且可能很多地方都用的上，所以可以抽取这些sql出来进行复用。

```xml
<sql id="selectColumn">
    id,last_name,email,gender
</sql>

<select id="testSql" resultType="top.tomxwd.mybatis.bean.Employee">
    select 
    <include refid="selectColumn"></include>
    from tbl_employee;
</select>
```

如此使用。

---

还可以使用_databaseId与if标签结合，动态变化sql的形式。

```xml
<sql id="selectColumn">
    <if test="_databaseId=='oracle'">
        employee_id,emp_last_name,emp_email,emp_gender
    </if>
    <if test="_databaseId=='oracle'">
        id,last_name,email,gender
    </if>
</sql>
```

---

**可以使用在select以及insert等场景，总之复用多的就可以抽取出来用，在需要的地方用include标签即可**

## include标签自定义属性

比如说，想要传递一些值过去，控制sql片段的内容（动态加列等），可以这样做，例子如下：

```xml
<select id="testSql" resultType="top.tomxwd.mybatis.bean.Employee">
    select
    <include refid="selectColumn">
        <property name="testColumn" value="abc"/>
    </include>
    from tbl_employee;
</select>

<sql id="selectColumn">
    id,last_name,email,gender,${abc}
</sql>
```

**注意：**

这里要用$符来去值，不可以用#来取。