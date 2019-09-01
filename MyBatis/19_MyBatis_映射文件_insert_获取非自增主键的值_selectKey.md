---
title: 19_MyBatis_映射文件_insert_获取非自增主键的值_selectKey
date: 2019-08-10 00:27:13
tags: 
 - Java
 - Mybatis
categories:
 - Mybatis
---

# 19_MyBatis\_映射文件\_insert_获取非自增主键的值\_selectKey

mysql跟oracle都可以用`<selectKey>`这种方法，道理一样。

## 运行顺序分析

### Order为before：

```xml
<insert id="addEmp" databaseId="oracle">
    <selectKey keyProperty="id" order="BEFORE" resultType="Integer">
        <!-- 编写查询主键的sql语句 -->
        select EMPLOYEE_SEQ.nextval from dual;
    </selectKey>
    insert into tbl_employee (last_name,email,gender) values (#{lastName},#{email},#{gender})
</insert>
```

1. order为before，运行`selectKey`查询id的sql先；
   - 这个操作是将id值查出，然后封装给javaBean的对应属性。
2. 再运行插入sql；
   - 这个时候可以取出上一步封装进去的javaBean属性。

### Order为after：

```xml
<insert id="addEmp" databaseId="oracle">
    <selectKey keyProperty="id" order="AFTER" resultType="Integer">
        <!-- 编写查询主键的sql语句 -->
        select EMPLOYEE_SEQ.currval from dual;
    </selectKey>
    insert into tbl_employee (id,last_name,email,gender) values (employee_seq.nextval,#{lastName},#{email},#{gender})
</insert>
```

1. 先直接运行插入的sql，且sql的id是employee_seq.nextval
2. 这时候再执行查当前序列操作，即得到employee_seq.currval