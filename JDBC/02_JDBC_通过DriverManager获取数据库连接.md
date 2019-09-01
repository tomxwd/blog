---
title: 02_JDBC_通过DriverManager获取数据库连接
date: 2019-08-21 13:46:50
tags: 
 - Java
categories:
 - JDBC
---

# 02_JDBC_通过DriverManager获取数据库连接

DriverManager是驱动的管理类。

## 流程

1. 准备连接数据库的四个字符串

   - driverClass：驱动的全类名
   - jdbcUrl：JDBC URL
   - user：用户名
   - password：密码

2. 读取配置文件（properties），也可以直接写死；

3. 加载数据库驱动程序(注册驱动)

   - DriverManager.registerDriver(Class.forName(driverClass).newInstance())

   - Class.forName(driverClass)

   为什么可以直接用第二种方式，也可以实现加载数据库驱动呢？

   ```java
   public class Driver extends NonRegisteringDriver implements java.sql.Driver {
       //
       // Register ourselves with the DriverManager
       //
       static {
           try {
               java.sql.DriverManager.registerDriver(new Driver());
           } catch (SQLException E) {
               throw new RuntimeException("Can't register driver!");
           }
       }
   ```

   因此只用写Class.forName(driverClass)把驱动类加载到JVM中即可，加载初始化的时候回执行静态代码块，然后注册一个Driver进DriverManager。

4. 通过DriverManager的getConnection()方法获取数据库连接；



## 测试

测试使用DriverManager来获取连接：

代码：

```java
@Test
public void testDriverManager() throws Exception{
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
    // 加载数据库驱动程序
    Class.forName(driverClass);
    Connection connection = DriverManager.getConnection(jdbcUrl, user, password);
    System.out.println("connection = " + connection);
}
```

使用DriverManager的好处是，我们可以一次性注入多个Driver进去，在获取连接的时候，传入不同的参数即可切换不同的数据库驱动（这里其实是根据url的规则来找到的）。

好处：

1. 可以通过重载的getConnection方法获取数据库连接，较为方便。
2. 可以同时管理多个驱动程序：若注册了多个数据库驱动，则调用getConnection()方法时，传入的参数不同，即返回不同的数据库连接。

编写通用的获取连接方法：

代码：

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

