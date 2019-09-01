---
title: 05_JDBC_小结1(复习)
date: 2019-08-21 17:14:57
tags: 
 - Java
categories:
 - JDBC
---

# 05_JDBC_小结1(复习)

## API总结

- java.sql.**DriverManager**用来装载驱动程序，获取数据库连接；
- java.sql.**Connection**完成对某一指定数据库的连接；
- java.sql.**Statement**在一个给定的连接中作为SQL执行声明的容器，他包含了两个重要的子类型：
  - java.sql.**PreparedStatement**用于执行预编译的sql声明；
  - java.sql.**CallableStatement**用于执行数据库中存储过程的调用；
- java.sql.**ResultSet**对于给定声明取得结果的途径；



## 测试

### 准备环境

导入依赖：

```xml
<dependencies>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>5.1.47</version>
    </dependency>
</dependencies>
```



### 获取数据库连接对象：

版本1：

只是获取连接，而且写死连接属性配置。

代码：

```java
@Test
public void testGetConnection() throws Exception {
    // 1. 准备4个连接需要用到的字符串 driverClass,jdbcUrl,user,password
    String driverClass = "com.mysql.jdbc.Driver";
    String jdbcUrl = "jdbc:mysql://tomxwd.top:12389/test";
    String user = "root";
    String password = "root";
    // 2. 加载驱动 Class.forName(driverClass)
    Class.forName(driverClass);
    // 3. 调用DriverManager.getConnection(jdbcUrl,user,password)方法获取数据库连接
    Connection connection = DriverManager.getConnection(jdbcUrl, user, password);
    System.out.println("connection = " + connection);
}
```

版本2：

解决硬编码问题，编写jdbc.properties文件，然后测试连接。

jdbc.properties

```properties
driverClass=com.mysql.jdbc.Driver
jdbcUrl=jdbc:mysql://tomxwd.top:12389/test
user=root
password=root
```

代码：

```java
@Test
public void testGetConnection() throws Exception {
    // 0. 读取jdbc.properties配置文件
    /**
     * properties属性文件对应java中的Properties类
     * 可以使用类加载器来加载这个配置文件
     */
    Properties properties = new Properties();
    properties.load(this.getClass().getClassLoader().getResourceAsStream("jdbc.properties"));
    // 1. 准备4个连接需要用到的字符串 driverClass,jdbcUrl,user,password
    String driverClass = properties.getProperty("driverClass");
    String jdbcUrl = properties.getProperty("jdbcUrl");
    String user = properties.getProperty("user");
    String password = properties.getProperty("password");
    // 2. 加载驱动 Class.forName(driverClass)
    Class.forName(driverClass);
    // 3. 调用DriverManager.getConnection(jdbcUrl,user,password)方法获取数据库连接
    Connection connection = DriverManager.getConnection(jdbcUrl, user, password);
    System.out.println("connection = " + connection);
}
```

也可以试试Oracle的配置（驱动依赖jar以及jdbc.properties配置需要修改）。



### Statement测试

先抽取上一步的获取连接方法：

```java
public Connection getConnection() throws IOException, ClassNotFoundException, SQLException {
    // 0. 读取jdbc.properties配置文件
    /**
     * properties属性文件对应java中的Properties类
     * 可以使用类加载器来加载这个配置文件
     */
    Properties properties = new Properties();
    properties.load(this.getClass().getClassLoader().getResourceAsStream("jdbc.properties"));
    // 1. 准备4个连接需要用到的字符串 driverClass,jdbcUrl,user,password
    String driverClass = properties.getProperty("driverClass");
    String jdbcUrl = properties.getProperty("jdbcUrl");
    String user = properties.getProperty("user");
    String password = properties.getProperty("password");
    // 2. 加载驱动 Class.forName(driverClass)
    Class.forName(driverClass);
    // 3. 调用DriverManager.getConnection(jdbcUrl,user,password)方法获取数据库连接
    return DriverManager.getConnection(jdbcUrl, user, password);
}
```

再编写释放资源的方法：

```java
public void release(Statement statement,Connection connection){
    if(statement!=null){
        try {
            statement.close();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
    if(connection!=null){
        try {
            connection.close();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}

public void release(ResultSet resultSet,Statement statement,Connection connection){
    if(resultSet!=null){
        try {
            resultSet.close();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
    release(statement,connection);
}
```

编写更新数据的代码测试：

```java
/**
 * Statement是用于操作SQL的对象
 * @throws Exception
 */
@Test
public void testStatementUpdate() throws Exception{
    Connection connection = null;
    Statement statement = null;
    try {
        // 1. 获取数据库连接
        connection = getConnection();
        // 2. 调用connection对象的createStatement()方法获取Statement对象
        statement = connection.createStatement();
        // 3. 准备SQL语句
        String sql = "insert into customers (name,email,birth) values ('ttt','1dzts@qq.com','2014-04-25')";
        // 4. 发送SQL语句：调用Statement对象的executeUpdate(sql)方法
        statement.executeUpdate(sql);
    } catch (Exception e) {
        e.printStackTrace();
    } finally {
        // 5. 关闭数据库资源：由里向外关闭
        release(statement,connection);
    }
}
```

编写查询数据的代码测试：

```java
/**
 * 1. ResultSet封装JDBC的查询结果
 * @throws Exception
 */
@Test
public void testStatementSelect() throws Exception{
    Connection connection = null;
    Statement statement = null;
    ResultSet resultSet = null;
    try{
        connection = getConnection();
        statement = connection.createStatement();
        String sql = "Select id,name,email,birth from customers";
        // 调用Statement的executeQuery(sql)方法获取结果集
        resultSet = statement.executeQuery(sql);
        // 处理结果集,调用resultSet的next方法查看下一条记录是否有效 有效则继续下移
        while(resultSet.next()){
            // 使用getXxx()方法获取具体值，参数可以是索引，也可以是sql上的别名
            System.out.println("-----------------------------");
            int id = resultSet.getInt("id");
            String name = resultSet.getString("name");
            String email = resultSet.getString("email");
            Date birth = resultSet.getDate("birth");
            System.out.println("id = " + id);
            System.out.println("name = " + name);
            System.out.println("email = " + email);
            System.out.println("birth = " + birth);
        }
    } catch (Exception e){
        e.printStackTrace();
    } finally {
        release(resultSet,statement,connection);
    }
}
```



