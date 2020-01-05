---
title: 04_MySQL_DDL基本语法
date: 2020-01-05 10:15:57
tags: 
 - MySQL
categories:
 - MySQL
---

# 04_MySQL_DDL基本语法

>[相关代码GitHub仓库](https://github.com/tomxwd/MySQL-Basic)
>
>git@github.com:tomxwd/MySQL-Basic.git
>
>https://github.com/tomxwd/MySQL-Basic.git

DDL（Data Define Language）：数据定义语言，库和表的管理

1. 库的管理
   - 创建
   - 修改
   - 删除
2. 表的管理
   - 创建
   - 修改
   - 删除

3. 关键字
   - 创建：create
   - 修改：alter
   - 删除：drop



## 库的管理

### 库的创建

**语法：**

create database 【if not exists】 库名；

```sql
# 库的管理
# 1. 库的创建
# 案例： 创建库books
create database books;
create database if not exists books;
show create database books;
```



### 查看库的信息

**语法：**

show create database 库名;



### 库的修改

**语法：**

alter database 库名 属性 set 值；

```sql
# 2. 库的修改
# 案例：更改库的字符集
alter database books character set gbk;
show create database books;
```



### 库的删除

**语法：**

drop database 【if exists】 库名；

```sql
# 3. 库的删除
drop database books;
drop database if exists books;
```



## 表的管理

### 表的创建

**语法：**

create table 【if not exists】 表名(

​	列名 列的类型【(长度)，约束】,

​	列名 列的类型【(长度)，约束】,

​	.....

​	列名 列的类型【(长度)，约束】

)

```sql
# 表的管理
use books;
# 1. 表的创建
# 创建book书籍表
create table book
(
    id          int,
    bName       varchar(20),
    price       double,
    authorId    int,
    publishDate datetime
);
desc book;
# 创建author作者表
create table author
(
    id      int,
    au_name varchar(20),
    nation  varchar(10)
);
desc author;
```



### 表的修改

1. 修改列名
2. 修改列的类型或约束
3. 添加新的列
4. 删除列
5. 修改表名

**语法：**

alter table 表名 add|drop|modify|change column 列名 列类型 约束；

```sql
# 2.表的修改
# ① 修改列名
alter table book
    change column publishDate pubDate datetime;
# ② 修改列的类型或约束
alter table book
    modify column pubDate timestamp;
# ③ 添加新的列
alter table author
    add column annual double;
# ④ 删除列
alter table author
    drop column annual;
# ⑤ 修改表名
alter table authors rename to bok_authors;
alter table bok_authors rename to authors;
show tables;
```



### 表的删除

**语法：**

drop table 【if exists】 表名；

```sql
# 3. 表的删除
drop table authors;
show tables;
```



### 表的复制

#### 仅仅复制表的结构

**语法：**

create table 复制后表名 like 表名;

```sql
# 4. 表的复制
insert into author (id, au_name, nation)
values (1, '村上春树', '日本'),
       (2, '莫言', '中国'),
       (3, '冯唐', '中国'),
       (4, '金庸', '中国');
# ①：仅仅复制表的结构
create table author_copy like author;
```



#### 复制表的结构以及数据

**语法：**

create table 复制后的表名 【查询语句】;

1. 复制表的所有结构与数据
2. 复制表的结构与部分数据
3. 复制表的部分结构

```sql
# ②：复制表的结构+数据
create table author_copy2
select *
from author_copy2;
# 复制部分列与部分数据
create table author_copy3
select id, au_name
from author
where nation = '中国';
```









