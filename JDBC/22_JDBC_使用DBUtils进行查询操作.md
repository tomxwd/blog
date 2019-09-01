---
title: 22_JDBC_使用DBUtils进行查询操作
date: 2019-08-23 16:56:47
tags: 
 - Java
categories:
 - JDBC
---

# 22_JDBC_使用DBUtils进行查询操作

## QueryRunner

QueryRunner的query方法的返回值取决于其ResultSetHandler参数的handle方法的返回值，下面是流程详解（看源码就知道了）：

![QueryRunner](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/JDBC/22QueryRunner.png)



自定义结果集处理器：

```java
/**
 * 自定义ResultSetHandler
 */
class MyResultSetHandler implements ResultSetHandler{
    public Object handle(ResultSet resultSet) throws SQLException {
        List<Customer> customers = new ArrayList<Customer>();
        while (resultSet.next()){
            int id = resultSet.getInt(1);
            String name = resultSet.getString(2);
            String email = resultSet.getString(3);
            Date birth = resultSet.getDate(4);
            Customer customer = new Customer(id, name, email, birth);
            customers.add(customer);
        }
        return customers;
    }
}
```

编写查询测试方法：

```java
/**
 * QueryRunner的query方法的返回值取决于其ResultSetHandler参数的handle方法的返回值
 */
@Test
public void testQueryRunnerQuery(){
    Connection conn = null;
    QueryRunner queryRunner = new QueryRunner();
    String sql = "select * from customers";
    try {
        conn = JDBCTools.getConnection();
        // 最后一个为结果集处理器
        Object query = queryRunner.query(conn, sql, new MyResultSetHandler());
        System.out.println("query = " + query);
    } catch (IOException e) {
        e.printStackTrace();
    } catch (ClassNotFoundException e) {
        e.printStackTrace();
    } catch (SQLException e) {
        e.printStackTrace();
    } finally {
        JDBCTools.release(null,conn);
    }
}
```



## BeanHandler使用

把结果集的第一条记录转为创建BeanHandler对象时传入的Class参数对应的对象。

测试方法：

```java
@Test
public void testBeanHandler(){
    Connection conn = null;
    QueryRunner queryRunner = new QueryRunner();
    try {
        conn = JDBCTools.getConnection();
        String sql = "select id,name,email,birth from customers where id =?";
        Customer query = queryRunner.query(conn, sql, new BeanHandler<Customer>(Customer.class), 2);
        System.out.println("query = " + query);
    } catch (IOException e) {
        e.printStackTrace();
    } catch (ClassNotFoundException e) {
        e.printStackTrace();
    } catch (SQLException e) {
        e.printStackTrace();
    } finally {
        JDBCTools.release(null,conn);
    }
}
```

注意：如果查询到多条，仍然是只返回第一条，因为源码上是只对结果集next一次，所以也不会报错。



## BeanListHandler使用

返回多条记录，使用方法跟BeanHandler差不多；

测试方法：

```java
@Test
public void testBeanListHandler(){
    Connection conn = null;
    QueryRunner queryRunner = new QueryRunner();
    try {
        conn = JDBCTools.getConnection();
        String sql = "select id,name,email,birth from customers";
        List<Customer> list = queryRunner.query(conn, sql, new BeanListHandler<Customer>(Customer.class));
        for (Customer customer : list) {
            System.out.println("customer = " + customer);
        }
    } catch (IOException e) {
        e.printStackTrace();
    } catch (ClassNotFoundException e) {
        e.printStackTrace();
    } catch (SQLException e) {
        e.printStackTrace();
    } finally {
        JDBCTools.release(null,conn);
    }
}
```



## MapHandler使用

结果集返回一个Map对象，注意返回的key是列名而不是别名，底层用的是getColumnName而不是getColumnLabel；

测试方法：

```java
@Test
public void testMapHandler(){
    Connection conn = null;
    QueryRunner queryRunner = new QueryRunner();
    try {
        conn = JDBCTools.getConnection();
        String sql = "select id,name,email,birth from customers where id = ?";
        Map<String, Object> map = queryRunner.query(conn, sql, new MapHandler(), 2);
        System.out.println("map = " + map);
    } catch (IOException e) {
        e.printStackTrace();
    } catch (ClassNotFoundException e) {
        e.printStackTrace();
    } catch (SQLException e) {
        e.printStackTrace();
    } finally {
        JDBCTools.release(null,conn);
    }
}
```



## MapListHandler使用

返回一个`List<Map>`对象，也就是多条记录；

测试方法：

```java
@Test
public void testMapListHandler(){
    Connection conn = null;
    QueryRunner queryRunner = new QueryRunner();
    try {
        conn = JDBCTools.getConnection();
        String sql = "select id,name,email,birth from customers";
        List<Map<String, Object>> list = queryRunner.query(conn, sql, new MapListHandler());
        for (Map<String, Object> map : list) {
            System.out.println("map = " + map); 
        }
    } catch (IOException e) {
        e.printStackTrace();
    } catch (ClassNotFoundException e) {
        e.printStackTrace();
    } catch (SQLException e) {
        e.printStackTrace();
    } finally {
        JDBCTools.release(null,conn);
    }
}
```



## ScalarHandler使用

返回单个值；

测试代码：

```java
@Test
public void testScalarHandler(){
    Connection conn = null;
    QueryRunner queryRunner = new QueryRunner();
    try {
        conn = JDBCTools.getConnection();
        String sql = "select name from customers where id = ?";
        String name = queryRunner.query(conn, sql, new ScalarHandler<String>(), 2);
        System.out.println("name = " + name);
        sql = "select count(*) from customers";
        Long count = queryRunner.query(conn, sql, new ScalarHandler<Long>());
        System.out.println("count = " + count);
    } catch (IOException e) {
        e.printStackTrace();
    } catch (ClassNotFoundException e) {
        e.printStackTrace();
    } catch (SQLException e) {
        e.printStackTrace();
    } finally {
        JDBCTools.release(null,conn);
    }
}
```

