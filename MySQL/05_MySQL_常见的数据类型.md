---
title: 05_MySQL_常见的数据类型
date: 2020-01-11 16:02:26
tags: 
 - MySQL
categories:
 - MySQL
---

# 05_MySQL_常见的数据类型

数值型：

- 整型
- 小数
  - 定点数
  - 浮点数

字符型：

- 较短的文本：char、varchar
- 较长的文本：text、blob（较长的二进制数据）

日期型



## 数值型——整型

| 整数类型     | 字节 | 范围                                                         |
| ------------ | ---- | ------------------------------------------------------------ |
| tinyint      | 1    | 有符号：-128~127<br />无符号：0~255                          |
| smallint     | 2    | 有符号：-32768~32767<br />无符号：0~65535                    |
| mediumint    | 3    | 有符号：-8388608~8388607<br />无符号：0~1677215              |
| int、integer | 4    | 有符号：-2147483648~2147483647<br />无符号：0~4294967295     |
| bigint       | 8    | 有符号：-9223372036854775808~9223372036854775807<br />无符号：0~9223372036854775807*2+1 |



### 如何设置有符号和无符号

```sql
# 1.如何设置无符号和有符号
drop table if exists tab_int;
create table tab_int(
    t1 int,
    t2 int unsigned
);
desc tab_int;
```

- `desc table`会显示type为int unsigned；

- 默认有符号；
- 如果插入的数值超出了范围，会报错，而且**插入的是临界值**；
- 如果不设置长度，会有默认的长度，这里设置的长度，指的是如果数值不够这个长度，会自动补零，默认是看不出来的，可以在创建表的时候，字段添加zerofill属性，而且会导致添加unsigned（无符号）属性，这时候不够长度就会显示补0；（**长度表示最大宽度**）



## 数值型——小数

**分类：**

1. 浮点型

   | 浮点数类型  | 字节 | 范围                                              |
   | ----------- | ---- | ------------------------------------------------- |
   | float(M,D)  | 4    | ±1.75494351E-38~±3.402823466E+38                  |
   | double(M,D) | 8    | ±2.2250738585072014E-308~±1.7976931348623157E+308 |

2. 定点型

   | 定点数类型                 | 字节 | 范围                                                         |
   | -------------------------- | ---- | ------------------------------------------------------------ |
   | DEC(M,D)<br />DECTMAL(M,D) | M+2  | 最大取值范围与double相同，给定decimal的有效取值范围由M和D决定 |

**M和D：**

- M：整数部位+小数部位（长度）
- D：小数部位

如果超过范围，则插入临界值

**特点：**

1. M和D可以省略，如果省略，默认如下：

   | 类型    | 默认值 |
   | ------- | ------ |
   | float   | 无     |
   | double  | 无     |
   | decimal | (10,0) |

**对比：**

- 定点型的精确度比较高，如果要求插入数值的精度较高，如货币运算则考虑使用；

```sql
# 二、小数
/*
     1. 浮点型
     - float
     - double
     2. 定点型
     - DEC(M,D) / DECTMAL(M,D)
 */

create table tab_float (
    d1 float,
    d2 double,
    d3 decimal,
    f1 float (5,2),
    f2 double(5,2),
    f3 decimal(5,2)
);

desc tab_float;
# 测试插入数据
insert into tab_float (f1, f2, f3) values (1230.00,124.153,133.343);
# 查询表数据
select *
from tab_float;
```



## 字符型（还包括文本型、二进制）

**较短的文本：**

| 字符串类型 | 最多字符数           | 描述及存储需求     |
| ---------- | -------------------- | ------------------ |
| char(M)    | M（省略不填默认为1） | M为0~255之间的数   |
| varchar(M) | M（不可以省略）      | M为0~65535之间的数 |

char代表固定长度的字符，varchar则代表可变长度的字符

- char和varchar对比：
  - char空间耗费比较大，varchar节省空间
  - char的性能更好，而varchar性能低一点



- 枚举ENUM

不区分大小写

```sql
# 枚举Enum
create table tab_char
(
    c1 ENUM ('a','b','c')
);

insert into tab_char
values ('a');
insert into tab_char
values ('b');
insert into tab_char
values ('c');
insert into tab_char
values ('m');
insert into tab_char
values ('A');

select *
from tab_char;
```



- 集合SET

```sql
# 集合SET（不区分大小写）
create table tab_set
(
    s1 set ('a','b','c','d')
);

insert into tab_set
values ('a');
insert into tab_set
values ('a,b');
insert into tab_set
values ('a,b,d');
insert into tab_set
values ('a,b,d');
insert into tab_set
values ('a,b,e');

select *
from tab_set;
```



- 其他的：
  - binary和varbinary用于保存较短的二进制
  - enum保存枚举
  - set保存集合



**较长的文本：**

- text

**较大的二进制：**

- blob



## 日期型

| 日期和时间类型 | 字节 | 最小值              | 最大值              |
| -------------- | ---- | ------------------- | ------------------- |
| date           | 4    | 1000-01-01          | 9999-12-31          |
| datetime       | 8    | 1000-01-01 00:00:00 | 9999-12-31 23:59:59 |
| timestamp      | 4    | 19700101080001      | 2038年的某个时刻    |
| time           | 3    | -838:59:59          | 838:59:59           |
| year           | 1    | 1901                | 2155                |

**区别：**

1. timestamp支持的时间范围较小，取值范围：1970010108001——2038年的某个时间，datetime的取值范围：1000-1-1——9999-12-31
2. timestamp和实际时区有关，更能反映实际的日期，而datetime则只能反映出插入时的当地市区
3. timestamp的属性受MySQL版本和SQLMode【语法格式】的影响很大



## 原则

所选择的类型越简单越好，能保存数值的类型越小越好









