---
title: 07_MySQL_事务
date: 2020-02-16 17:40:56
tags: 
 - MySQL
categories:
 - MySQL
---

# 07_MySQL_事务

TCL：Transaction Control Language 事务控制语言

**事务：**

一个或一组sql语句组成的一个执行单元，这个执行单元要么全部执行，要么全部不执行。

## MySQL中的存储引擎

1. 概念：在MySQL中的数据用各种不同的技术存储在文件（或内存）中；
2. 通过`show engines`来查看MySQL支持的存储引擎；
3. 在MySQL中用的比较多的存储引擎有：InnoDB，MyISAM，MEMORY等。其中InnoDB支持事务，而MyISAM、MEMORY等不支持事务；



## ACID属性

1. 原子性（Atomicity）

   原子性是指事务是一个不可分割的工作单位，事务中的操作要么都发生，要么都不发生；

2. 一致性（Consistency）

   事务必须使数据库从一个一致性状态变换到另一个一致性状态。

3. 隔离性（Isolation）

   事务的隔离性是指一个事务的执行不能被其他事务干扰，即一个事务内部的操作及使用的数据对并发的其他事务是隔离的，并发执行的各个事务之间不能互相干扰；

4. 持久性（Durability）

   持久性是指一个事务一旦被提交，它对数据库中数据的改变就是永久性的，接下来的其他操作和数据库故障不应该对其有任何影响；



## 事务创建

**隐式事务：**

- 事务没有明显的开启和结束的标记

- 比如insert、update、delete语句都属于隐式事务，
- 可以用`show variables like 'autocommit'`来查看是否为自动提交，ON为开启；

**显示事务：**

- 事务有明显的开启和结束的标记

- **前提：**必须设置自动提交（autocommit）为禁用

  `set autocommit = 0`

  只针对当前会话有效；

```sql
# 事务

# 查看自动提交事务
show variables like 'autocommit';

# 关闭自动提交事务
set autocommit = 0;

# 步骤1：开启事务 【可写可不写】
start transaction ;
# 步骤2：编写事务中的sql语句
# 语句1；
# 语句2；
# 步骤3：结束事务
# commit； 提交事务
# rollback；回滚事务
# savepoint 节点名；设置保存点
```



## 隔离级别（并发事务）

对于同时运行的多个事务，当这些事务访问**数据库中相同的数据**时，如果没有采取必要的隔离机制，就会导致各种并发问题；



用两个事务T1，T2来演示下面情形：

- 脏读：

  T1读取了已经被T2更新但是还**未提交**的字段之后，若T2回滚，T1读取到的内容就是临时且无效的；

- 不可重复读：

  T1读取了一个字段，然后**T2更新了该字段（已提交事务）**之后，T1再次读取同一个字段，值不同了；

- 幻读：

  T1从一个表中读取了一个字段，然后T2**在该表中插入（已提交事务）**了一些新的行，如果T1再次读取同一个表的数据，就会多出几行；

脏读跟幻读很像，但是脏读是针对更新，而幻读则针对插入或者删除，容易混淆；

**数据库事务的隔离性：**数据库系统必须具有隔离并发运行各个事务的能力，使它们不会相互影响，避免各种并发问题；

**一个事务与其他事务隔离的程度称为隔离级别**，数据库规定了多种事务隔离级别，不同隔离级别对应不同的干扰程度，隔离级别越高，数据一致性就越好，但是并发会越弱；



### 数据库提供的4中隔离级别

| 隔离级别                     | 描述                                                         |
| ---------------------------- | ------------------------------------------------------------ |
| READ UNCOMMITTED（读未提交） | 允许事务读取未被其他事务提交的变更，脏读，不可重复读，幻读的问题都会出现 |
| READ COMMITTED（读已提交）   | 只允许事务读取已经被其他事务提交的变更，可以避免脏读，但不可重复读和幻读仍然可能出现 |
| REPEATABLE READ（可重复读）  | 确保事务可以多次从一个字段中读取相同的值，在这个事务持续期间，禁止其他事务对这个字段进行更新，可以避免脏读和不可重复读，但是幻读的问题仍然存在 |
| SERIALIZABLE（串行化）       | 确保事务可以从一个表中读取相同的行，在这个事务持续期间，进制其他事务对该表执行插入、更新和删除操作，所有并发问题都可以避免，但是性能十分低下 |

- Oracle支持的2种事务隔离级别：READ COMMITTED，SERIALIZABLE。

  **默认是READ COMMITTED；**

- MySQL支持4中事务隔离级别，MySQL**默认的事务隔离级别是REPEATABLE READ**；



每启动一个mysql程序，就会获得一个单独的数据库连接。每个数据库连接都有一个全局变量@@tx_isolation，表示当前的事务隔离级别；

- 查看当前数据库默认隔离级别：

  `select @@tx_isolation;`

MySQL默认REPEATABLE READ；

- 设置隔离级别：

  `set session transaction isolation level read uncommitted`

  `set session transaction isolation level read committed`

  `set session transaction isolation level repeatable read`

  `set session transaction isolation level serializable`

  如果要测试隔离级别，先`set autocommit=0`关闭自动提交

- 设置数据库的全局隔离级别：

  `set global transaction isolation level read committed`

