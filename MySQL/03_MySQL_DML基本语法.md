---
title: 03_MySQL_DML基本语法
date: 2020-01-04 20:24:38
tags: 
 - MySQL
categories:
 - MySQL
---

# 03_MySQL_DML基本语法

DML语言：数据操作语言

- 插入：insert
- 修改：update
- 删除：delete



## 插入语句

**语法1：**

insert into 表名(列名,...) values (值1......);

1. 这个方法后面可以写多个行值，用逗号隔开；
2. 支持子查询插入，不写values关键字，在表名(列名)之后直接加select查询；



**语法2：**

insert into 表名 set 列名=值，列名=值，......



**注意：**

1. 插入的值的类型要与列的类型一致或兼容
2. 不可以为null的列必须要插入值，可以为null的列插入空值，或插入的时候不写该字段
3. 列的顺序可调换
4. 列数和值的个数必须一致
5. 可以省略列名，默认是所有列，而且列的顺序和表中列的顺序是一致的



**两种语法对比：**

1. 语法1支持插入多行，语法2不支持；
2. 语法1支持子查询，语法2不支持；

一般来讲使用方式1；



```sql
# 插入语句
use girls;

# 语法1
# 1. 插入的值的类型要与列的类型一致或兼容
insert into beauty(id, name, sex, borndate, phone, photo, boyfriend_id)
values (13, '唐艺昕', '女', '1990-4-23', '18988888888', null, 2);

# 2. 不可以为null的列必须要插入值，可以为null的列插入空值，或插入的时候不写该字段
# 方式1：
insert into beauty(id, name, sex, borndate, phone, photo, boyfriend_id)
values (13, '唐艺昕', '女', '1990-4-23', '18988888888', null, 2);
# 方式2：
insert into beauty(id, name, sex, borndate, phone, boyfriend_id)
values (14, '金星', '女', '1990-4-23', '18888888888', 9);

# 3. 列的顺序可调换
insert into beauty(name, sex, id, phone)
values ('蒋欣','女','16',110);

# 4. 列数和值的个数必须一致
# 5. 可以省略列名，默认是所有列，而且列的顺序和表中列的顺序是一致的
insert into beauty values (18,'张飞','男',NULL,'119',NULL,NULL);
# 语法1支持插入多行
insert into beauty values (19,'刘备','男',NULL,'1219',NULL,NULL),(20,'关羽','男',NULL,'12219',NULL,NULL);
# 语法1支持子查询插入数据
insert into beauty (id,name,phone) select 26,'宋茜','1181233';

# 语法2
insert into beauty
set id = 21,name='刘涛',phone='999';
```



## 修改语句

### 修改单表的记录

**语法：**

update 表名 set 列=新值,列=新值......

where 筛选条件（正常情况都要加条件）；



```sql
# 1. 修改单表的记录
# 案例1：修改beauty表中姓唐的女神的电话为13898999898
update beauty set phone = '13898999898' where name like '唐%';

# 案例2：修改boys表中id号为2的名称为张飞，魅力值为10
update boys set boyName = '张飞', userCP = 10 where id='2';
```



### 修改多表的记录【补充】

**92语法：**

update 表1 别名，表2 别名

set 列=值...

where 连接条件

and 筛选条件



**99语法：**

update 表1 别名

inner|left|right join 表2 b别名

on 连接条件

set 列=值......

where 筛选条件；



```sql
# 2. 修改多表的记录
# 案例1：修改张无忌的女朋友的手机号为114
update boys bo
inner join beauty b on bo.id=b.boyfriend_id
set b.phone='114'
where bo.boyName='张无忌';
# 案例2：没有男朋友的女神的男朋友编号为2
update beauty b
left join boys bo on bo.id = b.boyfriend_id
set b.boyfriend_id = 2
where bo.id is null;
```



## 删除语句

**语法一：**

delete from 表名 where 筛选条件



**语法二：**

truncate table 表名；（整个表的数据都删除）



### 删除单表的记录

```sql
# 方式1：delete
# 1. 单表的删除
# 案例1： 删除手机号以9结尾的女神信息
delete
from beauty
where phone like '%9';
```



### 删除多表的记录

sql92：

delete 【表1的别名】，【表2的别名】

from 表1 别名，表2 别名

where 连接条件

and 筛选条件；



sql99：

delete 【表1的别名】，【表2的别名】

from 表1 别名

inner|left|right join 表2 别名

on 连接条件

where 筛选条件；



```sql
# 2. 多表的删除
# 案例2：删除张无忌的女朋友的信息
delete b
from beauty b
         inner join boys bo on b.boyfriend_id = bo.id
where bo.boyName = '张无忌';
# 案例3：删除黄晓明的信息以及女朋友的信息
delete b,bo
from beauty b
         inner join boys bo on b.boyfriend_id = bo.id
where bo.boyName = '黄晓明';
```



### 清空表记录

```sql
# 方式2：truncate语句
# 案例：清空表数据
truncate table boys;
```



### delete和truncate对比

1. delete可以加where条件，truncate不能加；
2. truncate效率比delete高一点；
3. 假如要删除的表中有自增长列，如果delete删除后，再删除数据，自增长列的值从断点开始，而truncate删除后，再插入数据，自增长列的值从1开始；
4. truncate删除没有返回值，delete删除有返回值，返回受影响的行数；
5. truncate删除不能回滚，delete删除可以回滚；













