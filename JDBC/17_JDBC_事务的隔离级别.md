---
title: 17_JDBC_事务的隔离级别
date: 2019-08-23 11:14:18
tags: 
 - Java
categories:
 - JDBC
---

# 17_JDBC_事务的隔离级别

## 数据库的隔离级别

- 对于同时运行的多个事务，当这些事务访问数据库中相同的数据时，如果没有采取必要的隔离机制，就会导致各种并发问题：

  - **脏读**：对于两个事务T1，T2，如果T1读取了已经被T2更新但还**没有被提交**的字段，之后，若T2进行回滚，那么T1读取的内容就是临时且无效的。
  - **不可重复读**：对于两个事务T1，T2，事务T1读取了一个字段，然后T2**更新**了该字段，之后，T1再次读取同一个字段，值就不同了。
  - **幻读**：对于两个事务T1，T2，事务T1从一个表中读取了一个字段，然后T2在该表中**插入**了一些新的行，之后，如果T1再次读取同一个表，就会多出几行。

- 数据库事务的隔离性：数据库系统必须具有隔离并发运行各个事务的能力，使得它们不会相互影响，避免各种并发问题。

- **一个事务与其他事务隔离的程度称为隔离级别**，数据库规定了多种事务隔离级别，不同隔离级别对应不同的干扰程度，隔离级别越高，数据一致性就越好，但并发性越弱；

- 数据库提供了4中事务隔离级别：

  | 隔离级别                        | 描述                                                         |
  | ------------------------------- | ------------------------------------------------------------ |
  | READUNCOMMITTED（读未提取数据） | 允许事务读取未被其他事务提交的变更，脏读，不可重复读和幻读的问题都会出现； |
  | READCOMMITED（读已提交数据）    | 只允许事务读取已经被其他事务提交的变更，可以避免脏读，但不可重复读和幻读问题仍然可能出现； |
  | REPEATABLEREAD（可重复读）      | 确保事务可以多次从一个字段中读取相同的值，在这个事务持续期间，禁止其他事务对这个字段进行更新，可以避免脏读和不可重复度，但幻读的问题仍然存在。 |
  | SERIALIZABLE（串行化）          | 确保事务可以从一个表中读取相同的行，在这个事务持续期间，禁止其他事务对该表执行插入，更新和删除操作，所有并发问题都可以避免，但性能十分低下。 |

- Oracle支持的2种事务隔离级别：READCOMMITED和SERIALIZABLE；

  - Oracle默认的事务隔离级别为：READCOMMITED；

- MySQL支持4种事务隔离级别：

  - MySQL默认的事务隔离级别为：REPEATABLEREAD；



## JDBC的事务隔离级别控制

在JDBC程序中可以通过Connection的setTransationIsolation来设置隔离级别。

- Connection.TRANSACTION_NONE：没事务，值为0【MySql不支持，会报错】
- Connection.TRANSACTION_READ_UNCOMMITTED：读未提交，值为1
- Connection.TRANSACTION_READ_COMMITTED：读已提交，值为2
- Connection.TRANSACTION_REPEATABLE_READ：可重复读，值为4
- Connection.TRANSACTION_SERIALIZABLE：串行化，值为8

编写测试方法进行测试：

```java
/**
 * 测试事务的隔离级别
 * 在JDBC程序中可以通过Connection的setTransationIsolation来设置隔离级别。
 */
@Test
public void testTransaction3(){
    Connection conn = null;
    try {
        conn = JDBCTools.getConnection();
        // 设置不自动提交
        conn.setAutoCommit(false);
        // 设置隔离级别
        conn.setTransactionIsolation(Connection.TRANSACTION_READ_UNCOMMITTED);
        String sql = "update users set balance = balance - 500 where id = 1";
        JDBCTools.update(conn,sql);
        conn.commit();
    } catch (Exception e) {
        e.printStackTrace();
        try {
            conn.rollback();
        } catch (SQLException ex) {
            ex.printStackTrace();
        }
    } finally {
        JDBCTools.release(null,conn);
    }
}
```

这个时候可以在执行完update的地方打断点，然后去数据库查这个数据是否已经改变，或者再编写一个方法去查询。

可以发现在读未提交的时候，结果是很不可取的，没commit就可以读到。

MySql默认是4【TRANSACTION_REPEATABLE_READ】（不可重复读）。



## MySQL设置隔离级别

- 每启动一个mysql程序，就会获得一个单独的数据库连接，每个数据库连接都会有一个全局变量@@tx_isolation，表示当前的事务隔离级别。
- MySQL默认的隔离级别为RepeatableRead
- 查看当前的隔离级别：select @@tx_isolation；
- 设置当前MySQL连接的隔离级别：
  - set transaction isolation level read committed；
- 设置数据库系统的全局的隔离级别：
  - set global transaction isolation level read committed；

**注意：**

高版本要用`select @@transaction_isolation;`来查隔离级别；