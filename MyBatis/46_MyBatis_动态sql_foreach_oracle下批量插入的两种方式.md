---
title: 46_MyBatis_动态sql_foreach_oracle下批量插入的两种方式
date: 2019-08-11 22:22:45
tags: 
 - Java
 - Mybatis
categories:
 - Mybatis
---

# 46_MyBatis_动态sql_foreach_oracle下批量插入的两种方式

现在需要使用Oracle数据库进行批量保存，但是Oracle并不支持`values(),(),()`的形式，怎么办？

## 第一种方式

多条sql写在begin和end之间，即是

```sql
begin
	insert into xxx (Xxx,Xxx...) values(Xxx,Xxx...);
    insert into xxx (Xxx,Xxx...) values(Xxx,Xxx...);
    insert into xxx (Xxx,Xxx...) values(Xxx,Xxx...);
end;
```

## 第二种方式

利用中间表进行批量插入；

```sql
insert into employee(employy_id,last_name,email)
	select employee_seq.nextval,last_name,email from (
        select 'test_a_01' last_name,'test_a_e01' email from dual
        union
        select 'test_a_02' last_name,'test_a_e02' email from dual
        union
        select 'test_a_03' last_name,'test_a_e03' email from dual
    )
```



