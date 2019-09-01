---
title: 03_JDBC_通过Statement执行更新操作
date: 2019-08-21 14:29:45
tags: 
 - Java
categories:
 - JDBC
---

# 03_JDBC_通过Statement执行更新操作

## 访问数据库

- 数据库连接被用于向数据库服务器发送命令和SQL语句，在连接建立后，需要对数据库进行访问，执行sql语句；
- 在java.sql包中有3个接口分别定义了对数据库的调用的不同方式：
  - Statement
    - PreparedStatement
      - CallableStatement



## Statement

- 通过调用Connection对象的createStatement方法创建该对象。
- 该对象用于执行静态的SQL语句，并且返回执行结果；
- Statement接口中定义了下列方法用于执行SQL语句：
  - ResultSet executeQuery(String sql);
  - int executeUpdate(String sql);



## 测试

准备工作：

1. 创建test数据库

2. 创建customers表

   ```sql
   CREATE TABLE `customers` (
     `ID` int(6) NOT NULL AUTO_INCREMENT,
     `NAME` varchar(25) COLLATE utf8mb4_unicode_ci DEFAULT NULL,
     `EMAIL` varchar(25) COLLATE utf8mb4_unicode_ci DEFAULT NULL,
     `BIRTH` date DEFAULT NULL,
     PRIMARY KEY (`ID`)
   ) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
   ```

3. jdbc.properties

   ```properties
   driver=com.mysql.jdbc.Driver
   jdbcUrl=jdbc:mysql://Xxx:12389/test
   user=root
   password=root
   ```

4. 获取数据库连接方法：

   ```java
   public Connection getConnection2() throws Exception{
       // 1. 准备4个字符串参数
       String driverClass = null;
       String jdbcUrl = null;
       String user = null;
       String password = null;
       // 读取类路径下的properties文件
       InputStream in = this.getClass().getClassLoader().getResourceAsStream("jdbc.properties");
       Properties properties = new Properties();
       properties.load(in);
       driverClass = properties.getProperty("driver");
       jdbcUrl = properties.getProperty("jdbcUrl");
       user = properties.getProperty("user");
       password = properties.getProperty("password");
       // 2. 加载数据库驱动
       Class.forName(driverClass);
       // 3. 通过getConnection方法获取数据库连接
       Connection connection = DriverManager.getConnection(jdbcUrl, user, password);
       return connection;
   }
   ```

   

### 测试使用Statement插入一条数据

代码：

```java
/**
 * 通过JDBC向指定的数据表中插入一条记录
 */
@Test
public void testStatement() throws Exception {
    // 1. 获取数据库连接
    Connection conn = getConnection2();
    // 2. 准备插入的SQL语句
    String sql = "INSERT INTO customers (NAME,EMAIL,BIRTH) VALUES ('AXX!D','!!@bc@qq.com','2900-12-12');";
    // 3. 执行插入
    // 1) 获取操作SQL语句的Statement对象[调用Connection的createStatement方法]
    Statement statement = conn.createStatement();
    // 2) 调用Statement对象的executeUpdate(sql)执行sql语句进行插入
    statement.execute(sql);
    // 4. 关闭Statement对象
    statement.close();
    // 5. 关闭连接
    conn.close();
}
```

1. Statement：用于执行SQL语句的对象
   - 通过Connection的createStatement()方法来获取
   - 通过executeUpdate(sql)可以执行SQL语句
   - 传入的sql可以是INSERT，UPDATE或DELETE，但不能是SELECT，因为这里很明显是关于增删改的方法。
2. Statement和Connection都是应用程序和数据库服务器的连接资源，使用完后需要进行关闭。
   - 这里需要关注，如果在关闭资源之前出异常了怎么办，后面的代码并不执行，所以我们需要改进成try-catch-finally，然后finally中进行关闭资源操作。
3. 关闭的顺序是：先关闭后获取的，即先关闭Statement后关闭Connection；



### 对释放资源操作进行改进

代码：

```java
@Test
public void testStatement() throws Exception {
    // 1. 获取数据库连接
    Connection conn = null;
    Statement statement = null;
    try {
        conn = getConnection2();
        // 2. 准备插入的SQL语句
        String sql = "INSERT INTO customers (NAME,EMAIL,BIRTH) VALUES ('AXX!D','!!@bc@qq.com','2900-12-12');";
        // 3. 执行插入
        // 1) 获取操作SQL语句的Statement对象[调用Connection的createStatement方法]
        statement = conn.createStatement();
        // 2) 调用Statement对象的executeUpdate(sql)执行sql语句进行插入
        statement.execute(sql);
    } catch (Exception e) {
        e.printStackTrace();
    } finally {
        try {
            if(statement!=null){
                // 4. 关闭Statement对象
                statement.close();
            }
        } catch (SQLException e) {
            e.printStackTrace();
        } finally {
            if(conn!=null){
                // 5. 关闭连接
                conn.close();
            }
        }
    }

}
```

测试更新和删除操作，直接改sql语句即可，如：

```java
sql = "DELETE FROM customers WHERE ID = 1";
sql = "UPDATE customers set NAME='tom' WHERE ID=2";
```



## 写一个工具类管理资源，连接和释放

JBDCTool类代码：

```java
/**
 * 操作JDBC的工具类，其中封装了一些工具方法
 */
public class JDBCTool {

    /**
     * 1. 获取连接的方法
     * 通过读取配置文件从数据库服务器获取一个连接
     * @return
     * @throws Exception
     */
    public static Connection getConnection() throws Exception{
        // 1. 准备4个字符串参数
        String driverClass = null;
        String jdbcUrl = null;
        String user = null;
        String password = null;
        // 读取类路径下的properties文件
        InputStream in = JDBCTool.class.getClassLoader().getResourceAsStream("jdbc.properties");
        Properties properties = new Properties();
        properties.load(in);
        driverClass = properties.getProperty("driver");
        jdbcUrl = properties.getProperty("jdbcUrl");
        user = properties.getProperty("user");
        password = properties.getProperty("password");
        // 2. 加载数据库驱动
        Class.forName(driverClass);
        // 3. 通过getConnection方法获取数据库连接
        Connection connection = DriverManager.getConnection(jdbcUrl, user, password);
        return connection;
    }

    /**
     * 关闭Statement和Connection
     * @param statement
     * @param connection
     */
    public static void release(Statement statement,Connection connection){
        if(statement!=null){
            try {
                statement.close();
            } catch (Exception e){
                e.printStackTrace();
            }
        }
        if(connection!=null){
            try {
                connection.close();
            } catch (Exception e){
                e.printStackTrace();
            }
        }
    }

}
```



### 写一个通用的更新方法（增删改操作）

update方法代码：

```java
/**
 * 通用的更新方法：包括INSERT、UPDATE、DELETE
 * 第一个版本
 */
public void update1(String sql) throws Exception {
    Connection conn = null;
    Statement statement = null;
    try {
        conn = JDBCTool.getConnection();
        statement = conn.createStatement();
        statement.executeUpdate(sql);
    } catch (Exception e) {
        e.printStackTrace();
    } finally {
        JDBCTool.release(statement,conn);
    }
}
```

