---
title: 01_JDBC_通过Driver接口获取数据库连接
date: 2019-08-21 08:09:27
tags: 
 - Java
categories:
 - JDBC
---

# 01_JDBC_通过Driver接口获取数据库连接

## 数据持久化

- 持久化（persistence）：**把数据保存到可掉电式存储设备中以供之后使用**。大多数情况下，特别是企业级应用，数据持久化意味着将内存中的数据保存到硬盘上加以“固化”，而持久化的实现过程大多通过各种**关系数据库**来完成；
- 持久化的主要应用是将内存中的数据存储在关系型数据库中，当然也可以存储在磁盘文件、XML数据文件中；



## Java中的数据存储技术

- 在Java中，数据库存取技术可分为如下几类：
  - **JDBC直接访问数据库**
  - JDO技术
  - 第三方O/R工具，如Hibernate，ibatis等
- JDBC是Java访问数据库的基石，JDO，Hibernate等只是更好的封装了JDBC；



## JDBC基础

- JDBC（Java Database Connectivity）是一个**独立于特定数据库管理系统、通用的SQL数据库存取和操作的公共接口**（一组API），定义了用来访问数据库的标准Java类库，使用这个类库就可以以一种标准的方法、方便地访问数据库资源。
- JDBC为访问不同的数据库提供了一种统一的途径，为开发者屏蔽了一些细节问题；
- JDBC的目标是使Java程序员使用JDBC可以连接任何**提供了JDBC驱动程序**的数据库系统，这样就使得程序员无序对特定的数据库系统的特点有过多的了解，从而大大简化和加快了开发过程；

![关系图1](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/JDBC/01%E5%85%B3%E7%B3%BB%E5%9B%BE1.png)

![关系图2](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/JDBC/01%E5%85%B3%E7%B3%BB%E5%9B%BE2.png)



## JDBC体系结构

![JDBC体系结构](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/JDBC/01JDBC%E4%BD%93%E7%B3%BB%E7%BB%93%E6%9E%84.png)



## JDBC驱动程序分类

- JDBC驱动程序：各个数据库厂商根据JDBC的规范制作的JDBC实现库的类库；
- JDBC驱动程序总共有四种类型：
  - 第一类：JDBC-ODBC桥。
  - 第二类：部分本地API部分Java的驱动程序。
  - 第三类：JDBC网络纯Java驱动程序。
  - **第四类：本地协议的纯Java驱动程序**。
  - 第三、四两类都是纯Java驱动程序，因此对于Java开发者来说，它们在性能、可移植性、功能等方面都有优势。



### 本地协议的纯Java驱动程序

![本地协议的纯Java驱动程序](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/JDBC/01%E6%9C%AC%E5%9C%B0%E5%8D%8F%E8%AE%AE%E7%9A%84%E7%BA%AFJava%E9%A9%B1%E5%8A%A8%E7%A8%8B%E5%BA%8F.png)



## JDBC API

![JDBC-API](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/JDBC/01JDBC-API.png)



### Driver接口

- java.sql.Driver接口是所有JDBC驱动程序需要实现的接口。

  这个接口是提供给数据库厂商用的，不同数据库厂商提供不同的实现；

- 在程序中不需要直接去访问实现了Driver接口的类，而是由驱动程序管理器类（java.sql.DriverManager）去调用这些Driver实现；



## 加载与注册JDBC驱动

- 加载JDBC驱动需要调用Class类的静态方法forName()，向其传入需要加载的JDBC驱动的类名；
- DriverManager类是驱动程序管理器类，负责管理驱动程序；
- 通常不需要显式调用DriverManager类的registerDriver()方法来注册驱动程序类的实例，因为Driver接口的驱动程序类都包含了静态代码块，在这个静态代码块中，会调用DriverManager.registerDriver()方法来注册自身的一个实例；



## 建立连接

- 可以调用DriverManager类的getConnection()方法建立到数据库的连接；
- JDBC URL用于标识一个被注册的驱动程序，驱动程序管理器通过这个URL选择正确的驱动程序，从而建立到数据库的连接。
- JDBC URL的标准由三部分组成，各个部分间用冒号分隔：
  - **jdbc**:<子协议>:<子名称>
  - 协议：JDBC URL中的协议总是jdbc
  - 子协议：子协议用于标识一个数据库驱动程序
  - 子名称：一种标识数据库的方法。子名称可以依不同的子协议而变化，用子名称的目的是为了定位数据库提供足够的信息



## 几种常用数据库的JDBC URL

- 对于Oracle数据库连接，采用如下形式：
  - jdbc:oracle:thin:@localhost:1521:sid
- 对于SQLServer数据库连接，采用如下形式：
  - jdbc:microsoft:sqlserver//localhost:1433;DatabaseName=sid
- 对于MySQL数据库连接，采用如下形式：
  - jdbc:mysql://localhost:3306/sid



## 测试

Driver 是一个接口：数据库厂商必须实现的接口，能从其中获取数据库连接

pom.xml文件：

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

1. 加入MySql驱动
2. 创建一个Driver实现类的对象
3. 准备连接数据库的基本信息：url，user，password等。
4. 调用Driver接口的connect(url,info)方法获取数据库连接



### 测试连接：

```java
public class JDBCTest {

    @Test
    public void testDriver() throws Exception {
        Driver driver = new com.mysql.jdbc.Driver();
        String url = "jdbc:mysql://Xxx:3306/mybatis";
        Properties info = new Properties();
        info.put("user","root");
        info.put("password","root");
        Connection connetcion = driver.connect(url, info);
        System.out.println("connetcion = " + connetcion);
        connetcion.close();
    }

}
```

发现灵活性不高、耦合大，只能连接mysql，如果要用Oracle就需要重写一次流程。

**需求：**编写一个通用的方法，在不修改源程序的情况下，可以获取任何数据库的连接；

**解决方案：**把数据库驱动，Driver实现类的全类名、url、user、password放入一个配置文件中，通过修改配置文件的方式实现和具体的数据库解耦。

代码：

```java
public class JDBCTest {

    public Connection getConnection() throws Exception {
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
        // 用反射来创建实例，达到解耦
        Driver driver = (Driver) Class.forName(driverClass).newInstance();
        Properties info = new Properties();
        info.put("user",user);
        info.put("password",password);
        Connection connect = driver.connect(jdbcUrl, info);
        return connect;
    }

    @Test
    public void testConnection() throws Exception {
        Connection connection = getConnection();
        System.out.println("connection = " + connection);
    }

}
```

jdbc.properties

```java
driver=com.mysql.jdbc.Driver
jdbcUrl=jdbc:mysql://Xxxx:3306/test
user=root
password=root
```

如果要用Oracle数据库，就导入Oracle实现的驱动包，然后修改配置文件信息即可，其他数据库同理；

Oracle配置文件：

```properties
driver=oracle.jdbc.driver.OracleDriver
jdbcUrl=jdbc:oracle:thin:@Xxx:1521:test
user=root
password=root
```





