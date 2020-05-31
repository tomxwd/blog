---
title: 06_MySQL_常见的约束
date: 2020-01-13 21:57:52
tags: 
 - MySQL
categories:
 - MySQL
---

# 06_MySQL_常见的约束

**含义：**

- 一种限制，用于限制表中的数据，为了保证表中的数据的准确和可靠性



**分类：**

1. NOT NULL：非空，保证该字段的值不能为空
2. DEFAULT：默认，保证该字段有默认值
3. PRIMARY KEY：主键，保证该字段具有唯一性，并且**非空**
4. UNIQUE：唯一，用于保证该字段的值具有唯一性，可以为空
5. CHECK：检查，【MySQL中不支持，语法支持，但无效果】
6. FOREIGN KEY：外键，用于限制两个表的关系，用于保证该字段的值必须来自主表的关联列的值；



添加约束的时机：

- 创建表的时候
- 修改表的时候



约束的添加分类：

- 列级约束
  - 六大约束语法上都支持，但是外键约束没有效果；
- 表级约束
  - 除了非空、默认，其他的都支持



## 添加列级约束

直接在字段名和类型后面追加约束类型即可；

只支持：默认、非空、主键、唯一

```sql
# 常见的约束
create database if not exists students;
use students;
# 一、创建表时添加约束
# 1.添加列级约束
create table if not exists major
(
    id        int primary key,
    majorName varchar(20)
);

create table if not exists stuinfo
(
    id      int primary key,# 主键
    stuName varchar(20) not null,# 非空
    gender  char(1) check ( gender = '男' or gender = '女' ),# 检查
    seat    int unique,# 唯一
    age     int default 18,# 默认约束
    majorId int references major (id)#外键约束
);

desc stuinfo;
# 查看stuinfo表中所有的索引，包括主键、外键、唯一
show index from stuinfo;
```



## 添加表级约束

**语法：**

在各个字段的最下面：

【constraint 约束名 】约束类型（字段名）

不写约束名会有默认名；

```sql
# 2. 添加表级约束
drop table if exists stuinfo;
create table stuinfo
(
    id      int,
    stuname varchar(20),
    gender  char(1),
    seat    int,
    age     int,
    majorId int,
    constraint pk_id primary key (id),# 主键
    constraint uq_seat unique (seat),# 唯一键
    constraint ck_gender check ( gender = '男' or gender = '女' ),# 检查
    constraint fk_stuinfo_major foreign key (majorId) references major (id)# 外键
);

show index from students.stuinfo;
```

一般主键、唯一键、默认都用列级，外键用表级；



## 主键和唯一的区别

|                     | 保证唯一性 | 是否允许为空 | 一个表中可以有多少个列 | 是否允许组合 |
| ------------------- | ---------- | ------------ | ---------------------- | ------------ |
| 主键（primary key） | √          | ×            | 至多有一个             | √，但不推荐  |
| 唯一（unique）      | √          | √            | 可以有多个             | √，但不推荐  |



## 外键特点

1. 要求在从表设置外键关系；
2. 从表的外键列的类型和主表的关联列的类型要求一致或兼容，名称无要求；
3. 主表的关联列必须是一个key（一般是主键或唯一）；
4. 插入数据的时候，要先插入主表，再插入从表
5. 删除数据的时候，先删除从表，再删除主表



## 修改表的时候添加约束

1. 添加列级约束

   alter table 表名 modify column 字段名 字段类型 新约束;

2. 添加表级约束

   alter table 表名 add 【constraint 约束名】约束类型(字段名)【外键的引用 references 表名(字段名)】;

```sql
# 二、修改表时添加约束
drop table if exists stuinfo;
create table stuinfo
(
    id      int,
    stuname varchar(20),
    gender  char(1),
    seat    int,
    age     int,
    majorId int
);
desc students.stuinfo;
# 1. 添加非空约束
alter table students.stuinfo
    modify column stuname varchar(20) not null;
# 2. 添加默认约束
alter table students.stuinfo
modify column age int default 18;
# 3. 添加主键
# ①：列级约束
alter table students.stuinfo modify column id int primary key ;
# ②：表级约束
alter table students.stuinfo add primary key (id);
# 4. 添加唯一
# ①：列级约束
alter table students.stuinfo modify column seat int unique ;
# ②：表级约束
alter table students.stuinfo add unique (seat);
# 5. 添加外键
# mysql只有表级生效
alter table students.stuinfo add foreign key (majorId) references major(id);
# 指定外键名字
alter table students.stuinfo add constraint fk_stuinfo_major foreign key (majorId) references major(id);
```



## 修改表时删除约束

```sql
# 三、修改表时删除约束
# 1. 删除非空约束
alter table students.stuinfo modify column stuname varchar(20);
# 2. 删除默认约束
alter table students.stuinfo modify column age int;
# 3. 删除主键约束
alter table students.stuinfo modify column id int;
alter table students.stuinfo drop primary key ;
# 4. 删除唯一
alter table students.stuinfo drop index seat;
# 5. 删除外键
alter table students.stuinfo drop foreign key fk_stuinfo_major;
```



## 表级和列级约束对比

|              | 位置         | 支持的约束类型             | 是否可以起约束名   |
| ------------ | ------------ | -------------------------- | ------------------ |
| **列级约束** | 列的后面     | 语法都支持，但外键无效     | 不可以             |
| **表级约束** | 所有列的下面 | 默认和非空不支持，其他支持 | 可以（主键无效果） |



## 标识列（自增长列）

**含义：**

可以不用手动的插入值，系统提供默认的序列值；

**使用方式：**

1. 创建表的时候设置标识列，字段加 AUTO_INCREMENT即可；
2. 修改表的时候设置标识列；
3. 修改表的时候删除标识列；

**其他：**

可以用`show variables like '%auto_increment%'`来看有关自增长的设置，一个是步长（auto_increment_increment），一个是起始值【偏移量】（auto_increment_offset）；

可以用set auto_increment_increment=3来改变步长为3；

**特点：**

1. 标识列必须需要一个Key，主键或者UNIQUE等；
2. 一个表中只能有一个标识列；
3. 标识列的类型需要是数值型（int，float，double等）；
4. 标识列可以通过`set auto_increment_increment=3`来改变步长，也可以通过第一次插入指定值来改变起始值；
5. 

