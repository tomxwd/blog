---
title: 07_JDBC_PreparedStatement
date: 2019-08-22 10:49:43
tags: 
 - Java
categories:
 - JDBC
---

# 07_JDBC_PreparedStatement

拼sql语句是非常麻烦的，特别是字段越来越多的时候，还要考虑这个字段是什么类型，需不需要拼接单引号。所以我们使用PreparedStatement替代。

PreparedStatement是Statement的子接口，所以可以用到Statement的方法。可以传入带占位符的sql语句，并且提供了补充占位符变量的方法。

PreparedStatement还可以有效防止sql注入。



## SQL注入攻击

- SQL注入是利用某些系统没有对用户输入的数据进行充分的检查，而在用户输入数据中注入非法的SQL语句段或者命令，从而利用系统的SQL引擎完成恶意行为的做法；
- 对于Java而言，要防范SQL注入，只需要用PreparedStatement取代Statement就可以了；




## PreparedStatement

- 可以通过调用Connection对象的preparedStatement()方法获取PreparedStatement对象；
- PreparedStatement接口是Statement的子接口，它表示一条预编译过的SQL语句；
- PreparedStatement对象所代表的的SQL语句中的参数用问号(?)来表示，调用PreparedStatement对象的setXXX()方法来设置这些参数，setXXX()方法有两个参数，第一个参数是要设置的SQL语句中的参数的索引（从1开始），第二个是设置的SQL语句中的参数的值；



## PreparedStatement和Statement

- 代码的可读性和可维护性
- PreparedStatement能最大可能提高性能：
  - DBServer会对预编译语句提供性能优化。因为预编译语句有可能被重复调用，所以语句在被DBServer的编译器编译后的执行代码被缓存下来，那么下次调用的时候只是要相同的预编译语句就不需要进行编译了，只要将参数直接传入编译过的语句中，就会得到执行；
  - 在Statement语句中，即使是相同操作但是因为数据内容的不一样，所以整个语句本身不能匹配，没有缓存语句的意义。事实上是没有数据库会对普通语句编译后的执行代码缓存，这样每执行一次都要对传入的语句编译一次；
  - 语法检查、语义检查、翻译成二进制命令、缓存
- PreparedStatement可以防止SQL注入；



## PreparedStatement的使用

基本形式：

insert into eaxmstudent values(?,?,?,?,?,?,?)；

使用步骤：

由于PreparedStatement是Statement的子接口，所以释放资源操作不需要重载等，直接用Statement的就行。

1. 创建PreparedStatement对象，不过在创建这个对象的时候必须传入sql：

   PreparedStatement ps = conn.preparedStatement(sql);

2. 调用PreparedStatement对象的setXxx(int index,Object val)；设置占位符的值。

   **注意：索引值从1开始**；

3. 执行sql语句：

   - executeQuery()
   - executeUpdate()
   - 注意执行的时候不再需要传入sql语句

测试代码：

```java
@Test
public void testPreparedStatement(){
    Connection connection = null;
    PreparedStatement ps = null;
    try {
        connection = JDBCTools.getConnection();
        String sql = "insert into customers (name,email,birth)" +
            "values(?,?,?)";
        ps = connection.prepareStatement(sql);
        ps.setString(1,"tottom");
        ps.setString(2,"xsd@qq.com");
        ps.setDate(3,new Date(new java.util.Date().getTime()));
        ps.executeUpdate();
    } catch (IOException e) {
        e.printStackTrace();
    } catch (ClassNotFoundException e) {
        e.printStackTrace();
    } catch (SQLException e) {
        e.printStackTrace();
    } finally {
        JDBCTools.release(ps,connection);
    }
}
```

编写JDBCTools的通用update方法，用PreparedStatement实现：

```java
public static void update(String sql,Object ...args){
    Connection conn = null;
    PreparedStatement ps = null;
    try {
        conn = JDBCTools.getConnection();
        ps = conn.prepareStatement(sql);
        for (int i = 0; i < args.length; i++) {
            ps.setObject(i+1,args[i]);
        }
        ps.executeUpdate();
    } catch (IOException e) {
        e.printStackTrace();
    } catch (ClassNotFoundException e) {
        e.printStackTrace();
    } catch (SQLException e) {
        e.printStackTrace();
    }
}
```





