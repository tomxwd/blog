---
title: 16_JDBC_处理事务
date: 2019-08-23 10:22:47
tags: 
 - Java
categories:
 - JDBC
---

# 16_JDBC_处理事务

## 事务

- 在数据库中，所谓事务是指**一组逻辑操作单元，使数据从一种状态变换到另一种状态**。
- 为确保数据库中数据的**一致性**，数据的操纵应当是离散的成组的逻辑单元：当它全部完成时，数据的一致性可以保持，而当这个单元中的一部分操作失败，整个事务应全部视为错误，所有从起始点以后的操作应全部回退到开始状态。
- 事务的操作：先定义开始一个事务，然后对数据作修改操作，这时候如果**提交（COMMIT）**，这些修改就会永久得保存下来，如果**回退（ROLLBACK）**，数据库管理系统将放弃所作的所有修改而回退到开始事务时的状态。



## 事务的ACID属性

1. 原子性（Atomicity）

   原子性是指事务是一个不可分割的工作单位，事务中的操作要么都发生，要么都不发生。

2. 一致性（Consistency）

   事务必须使数据库从一个一致性状态变换到另外一个一致性状态。

3. 隔离性（Isolation）

   事务的隔离性是指一个事务的执行不能被其他事务干扰，即一个事务内部的操作以及使用的数据对并发的其他事务是隔离的，并发执行的各个事务之间不能互相干扰。

4. 持久性（Durability）

   持久性是指一个事务一旦被提交，它对数据库中数据的改变就是永久性的，接下来的其他操作和数据库故障不应该对其有任何影响。



## JDBC的事务处理

- 事务：指构成单个逻辑工作单元的操作集合；
- 事务处理：保证所有事务都作为一个工作单元来执行，即使出现了故常，都不能改变这种执行方式。当在一个事务中执行多个操作时，要么所有的事务都被提交（commit），要么整个事务回滚（rollback）到最初状态；
- 当一个连接对象被创建时，默认情况下是自动提交事务：每次执行一个SQL语句时，如果执行成功，就会向数据库自动提交，而不能回滚；
- 为了让多个SQL语句作为一个事务执行：
  - 调用Connection对象的setAutoCommit(false);以取消自动提交事务；
  - 在所有的SQL语句都成功执行后，调用commit();方法提交事务；
  - 在出现异常的时候，调用rollback();方法回滚事务；
  - 若此时Connection没有被关闭，则需要恢复其自动提交状态；



## 测试

### 准备工作：

数据库表：

```sql
CREATE TABLE `users` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `username` varchar(255) COLLATE utf8mb4_unicode_ci DEFAULT NULL,
  `password` varchar(255) COLLATE utf8mb4_unicode_ci DEFAULT NULL,
  `balance` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```



### 编写代码

首先编写一个测试方法，暴露问题所在：

```java
@Test
public void Transaction(){
    // 1. tom给jerry汇款500
    dao.update("UPDATE users set balance=balance-500 where id =1");
    int i = 10/0;
    dao.update("UPDATE users set balance=balance+500 where id =2");
}
```

这样只会更新第一条语句，原因是每次更新都是新的Connection；



分析：

1. 如果多个操作，每个操作使用的是自己单独的连接，则无法保证事务。



事务步骤：

1. 首先要自己管理连接。
2. 关闭自动提交
   - connection.setAutoCommit(false)
3. 执行完成提交事务
   - connection.commit()
4. 异常则回滚
   - connection.rollback()



首先重写工具类的update方法，传入Connection：

JDBCTools.update(Connection conn,String sql)

```java
public static void update(Connection conn,String sql){
    Statement statement = null;
    try {
        // 2. 调用connection对象的createStatement()方法获取Statement对象
        statement = conn.createStatement();
        // 3. 准备SQL语句
        // 4. 发送SQL语句：调用Statement对象的executeUpdate(sql)方法
        statement.executeUpdate(sql);
    } catch (Exception e) {
        e.printStackTrace();
    } finally {
        // 5. 关闭数据库资源：由里向外关闭
        release(statement,null);
    }
}
```

编写事务测试：

```java
@Test
public void Transaction2(){
    Connection conn = null;
    try {
        conn = JDBCTools.getConnection();
        // 开始事务
        // 取消默认提交
        conn.setAutoCommit(false);
        JDBCTools.update(conn,"UPDATE users set balance=balance-500 where id =1");
        int i = 10/0;
        JDBCTools.update(conn,"UPDATE users set balance=balance+500 where id =2");
        // 执行完毕，提交事务
        conn.commit();
    } catch (Exception e) {
        e.printStackTrace();
        // 出现异常回滚事务
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





