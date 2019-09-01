---
title: 18_MyBatis_映射文件_insert_Oracle使用序列生成主键演示
date: 2019-08-09 23:51:11
tags: 
 - Java
 - Mybatis
categories:
 - Mybatis
---

# 18_MyBatis\_映射文件_insert_Oracle使用序列生成主键演示

## 在Oracle中如何获取序列

- Oracle不支持自增，但是Oracle使用序列来模拟自增。

- 每次插入数据的主键是从序列中拿到的值，如何获取到值

  - 查看当前用户序列

    ```sql
    select * from user_sequences;
    ```

  - 获取序列的下一个值

    ```sql
    select EMPLOYEE_SEQ.nextval from dual;
    ```

    这样，每次执行都会进行递增。

## 在mybatis中如何获取

```xml
<insert id="addEmp" databaseId="oracle">
    <selectKey keyProperty="id" order="BEFORE" resultType="Integer">
        <!-- 编写查询主键的sql语句 -->
        select EMPLOYEE_SEQ.nextval from dual;
    </selectKey>
    insert into tbl_employee (last_name,email,gender) values (#{lastName},#{email},#{gender})
</insert>
```

主要就是在insert标签下加一个`<selectKey>`标签，然后编写获取下一个序列的sql语句。

`<selectKey>`

- 这个标签也有keyProperty属性需要设定，指定javabean的对应赋值属性。
- order属性
  - 值为BEFORE的时候，是在插入sql之前执行
  - 值为AFTER的时候，是在插入sql之后执行
- resultType属性
  - 返回值的类型