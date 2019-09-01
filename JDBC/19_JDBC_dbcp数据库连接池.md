---
title: 19_JDBC_dbcp数据库连接池
date: 2019-08-23 14:35:23
tags: 
 - Java
categories:
 - JDBC
---

# 19_JDBC_dbcp数据库连接池

## JDBC数据库连接池的必要性

- 在使用开发基于数据库的web程序时，**传统的模式**基本是按以下步骤：
  1. 在主程序（如servlet、beans）中建立数据库连接。
  2. 进行sql操作。
  3. 断开数据库连接。
- 这种开发模式，存在的问题：
  1. 普通的JDBC数据库连接使用DriverManager来获取，每次向数据库建立连接的时候都要将Connection加载到内存，再验证用户名和密码（得花费0.05s~1s的时间）。需要数据库连接的时候，就向数据库要求一个，执行完毕后再断开连接。这样的方式会小号大量的资源和时间。**数据库的连接资源并没有得到很好的重复利用**。若同时有上百人甚至上千人在线，频繁的进行数据库连接操作将占用很多的系统资源，严重的甚至会造成服务器的崩溃。
  2. 对于每一次数据库连接，使用完后都得断开。否则，如果程序出现异常而未能关闭，将会导致数据库系统的内存泄漏，最终将导致重启数据库。
  3. 这种开发不能控制被创建的连接对象数，系统资源会毫无顾及的分配出去，如连接过多，也可能导致内存泄漏，服务器崩溃。



## 数据库连接池（Connection Pool）

- 为了解决传统开发中数据库连接问题，可以采用数据库连接池技术。
- 数据库连接池的基本思想就是为**数据库连接建立一个“缓冲池”**，预先在缓冲池中放入一定数量的连接，当需要建立数据库连接时，只需从“缓冲池”中取出一个，使用完毕后再放回去。
- **数据库连接池**负责分配、管理和释放数据库连接，它**允许应用程序重复使用一个现有的数据库连接，而不是重新建立一个**；
- 数据库连接池在初始化时将创建一定数量的数据库连接放到连接池中，这些数据库连接的数量是由**最小数据库连接数来设定**的。无论这些数据库连接是否被使用，连接池都将一直保证至少拥有这么多的连接数量。连接池的**最大数据库连接数量**限定了这个连接池能战友的最大连接数。当应用程序向连接池请求的连接数超过最大连接数量时，这些请求将被加入到等待队列中。

![数据库连接池](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/JDBC/19%E6%95%B0%E6%8D%AE%E5%BA%93%E8%BF%9E%E6%8E%A5%E6%B1%A0.png)



## 数据库连接池技术的优点

1. 资源重用：
   - 由于数据库连接得以重用，避免了频繁的创建，释放连接引起的大量性能开销。
   - 在减少系统消耗的基础上，另一方面也增加了系统运行环境的平稳性。
2. 更快的系统反应速度：
   - 数据库连接池在初始化过程中，往往已经创建了若干数据库连接置于连接池中备用。此时连接的初始化工作均已完成。对于业务请求处理而言，直接利用现有可用连接。避免了数据库连接初始化和释放过程的时间开销，从而减少了系统的响应时间。
3. 新的资源分配手段：
   - 对于多应用共享同一数据库的系统而言，可在应用层通过数据库连接池的配置，实现某一应用最大可用数据库连接数的限制，避免某一应用独占所有的数据库资源。
4. 统一的连接管理，避免数据库连接泄漏：
   - 在较为完善的数据库连接池实现中，可根据预先的占用超时设定，强制回收被占用连接，从而避免了常规数据库连接操作中可能出现的资源泄漏。



## 两种开源的数据库连接池

- JDBC的数据库连接池使用javax.sql.DataSource来表示，DataSource只是一个接口，该接口通常是由服务器（Weblogic，WebSphere，Tomcat）提供实现，也有一些开源组织提供实现：
  - DBCP数据库连接池
    - DBCP是Apache软件基金组织下的开源连接池实现，该连接池依赖该组织下的另一个开源系统：common-pool。如需使用该连接池实现，应在系统中增加如下两个jar文件：
      - commons-dbcp.jar：连接池的实现
      - commons-pool.jar：连接池实现的依赖库
    - Tomcat的连接池正式采用该连接池来实现的，该数据库连接池即可以用来与应用服务器整合使用，也可以由应用程序单独使用。
  - C3P0数据库连接池
- DataSource通常被称为数据源，它包含连接池和连接池管理两个部分，习惯上也经常把DataSource称为连接池。



## 测试

先导入依赖：

这里不单单是commons-dbcp，还有commons-pool；

```xml
<dependency>
    <groupId>commons-dbcp</groupId>
    <artifactId>commons-dbcp</artifactId>
    <version>1.4</version>
</dependency>
```

创建数据库连接池并配置：

```java
/**
  * 测试DBCP连接池
  */
@Test
public void testDBCP() throws SQLException {
    // 1. 创建DBPC数据源实例
    BasicDataSource dataSource = new BasicDataSource();
    // 2. 为数据库实例指定必须的属性
    dataSource.setUsername("root");
    dataSource.setPassword("root");
    dataSource.setUrl("jdbc:mysql://tomxwd.top:12389");
    dataSource.setDriverClassName("com.mysql.jdbc.Driver");
    // 3. 指定数据源中一些可选的属性
    // 3.1 指定数据库连接池中初始化连接数的个数
    dataSource.setInitialSize(10);
    // 3.2 指定数据库最大的活动连接数
    dataSource.setMaxActive(50);
    // 3.3 指定最小的空闲数
    dataSource.setMinIdle(5);
    // 3.4 等待数据库连接池分配连接的最长时间，单位为毫秒，超出时间则抛出异常
    dataSource.setMaxWait(1000*5);
    // 4. 从数据源中获取数据库连接
    Connection conn = dataSource.getConnection();
    System.out.println("conn = " + conn);
    System.out.println("conn.getClass() = " + conn.getClass());
}
```

也可以用BasicDataSourceFactory来获取：

先编写一个配置文件（要对应上，否则报错）：

```properties
url=jdbc:mysql://tomxwd.top:12389/test
username=root
password=root
driverClassName=com.mysql.jdbc.Driver
initialSize=10
maxActive=50
minIdle=5
maxWait=5000
```

测试类：

```java
@Test
public void testDBCPWithDataSourceFactory() throws Exception {
    Properties properties = new Properties();
    properties.load(this.getClass().getClassLoader().getResourceAsStream("dbcp.properties"));
    DataSource dataSource = BasicDataSourceFactory.createDataSource(properties);
    Connection connection = dataSource.getConnection();
    System.out.println("connection = " + connection);
    BasicDataSource basicDataSource = (BasicDataSource) dataSource;
    System.out.println(basicDataSource.getMaxActive());
}
```

dbcp2以后，没有setMaxActive了，改用setMaxTotal；