---
title: 20_JDBC_C3P0数据库连接池
date: 2019-08-23 15:45:02
tags: 
 - Java
categories:
 - JDBC
---

# 20_JDBC_C3P0数据库连接池

引入c3p0依赖：

```xml
<dependency>
    <groupId>com.mchange</groupId>
    <artifactId>c3p0</artifactId>
    <version>0.9.5.4</version>
</dependency>
```

基本使用测试：

```java
@Test
public void testC3P0() throws PropertyVetoException, SQLException {
    ComboPooledDataSource cpds = new ComboPooledDataSource();
    cpds.setDriverClass("com.mysql.jdbc.Driver");
    cpds.setJdbcUrl("jdbc:mysql://tomxwd.top:12389/test");
    cpds.setUser("root");
    cpds.setPassword("root");
    Connection connection = cpds.getConnection();
    System.out.println("connection = " + connection);
}
```

官方推荐方式（使用xml配置文件形式）c3p0-config.xml：

[官网地址](https://www.mchange.com/projects/c3p0/#c3p0-config.xml)

```xml
<c3p0-config>
    <!--默认配置-->
<!--    <default-config>
        <property name="automaticTestTable">con_test</property>
        <property name="checkoutTimeout">30000</property>
        <property name="idleConnectionTestPeriod">30</property>
        <property name="initialPoolSize">10</property>
        <property name="maxIdleTime">30</property>
        <property name="maxPoolSize">100</property>
        <property name="minPoolSize">10</property>
        <property name="maxStatements">200</property>

        <user-overrides user="test-user">
            <property name="maxPoolSize">10</property>
            <property name="minPoolSize">1</property>
            <property name="maxStatements">0</property>
        </user-overrides>

    </default-config>-->

    <!-- This app is massive! -->
    <named-config name="helloc3p0">

        <!-- 指定连接数据库基本属性 -->
        <property name="user">root</property>
        <property name="password">root</property>
        <property name="jdbcUrl">jdbc:mysql://tomxwd.top:12389/test</property>
        <property name="driverClass">com.mysql.jdbc.Driver</property>

        <!-- 若数据库中连接数不足的时候，一次向数据库服务器申请多少个连接 -->
        <property name="acquireIncrement">50</property>
        <!-- 初始化数据库连接池时的连接数量 -->
        <property name="initialPoolSize">100</property>
        <!-- 数据库连接池中最小的连接数 -->
        <property name="minPoolSize">50</property>
        <!-- 数据库连接池中最大的连接数 -->
        <property name="maxPoolSize">1000</property>

        <!-- intergalactoApp adopts a different approach to configuring statement caching -->
        <!-- C3P0数据库连接池可以维护的Statement的个数 -->
        <property name="maxStatements">0</property>
        <!-- 每个连接可以使用的Statement对象的个数 -->
        <property name="maxStatementsPerConnection">5</property>

        <!-- 覆盖的属性 -->
        <!-- he's important, but there's only one of him -->
<!--        <user-overrides user="master-of-the-universe">
            <property name="acquireIncrement">1</property>
            <property name="initialPoolSize">1</property>
            <property name="minPoolSize">1</property>
            <property name="maxPoolSize">5</property>
            <property name="maxStatementsPerConnection">50</property>
        </user-overrides>-->
    </named-config>
</c3p0-config>
```

测试方法：

```java
@Test
public void testC3P0WithConfigFile() throws SQLException {
    // 这里写配置文件内指定的名字
    ComboPooledDataSource dataSource = new ComboPooledDataSource("helloc3p0");
    Connection connection = dataSource.getConnection();
    System.out.println("connection = " + connection);
    System.out.println("dataSource.getMaxStatements() = " + dataSource.getMaxStatements());
}
```

