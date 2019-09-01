---
title: 38_MyBatis_动态sql_简介&环境搭建
date: 2019-08-11 16:14:16
tags: 
 - Java
 - Mybatis
categories:
 - Mybatis
---

# 38_MyBatis\_动态sql_简介&环境搭建

> [官网---动态sql](http://www.mybatis.org/mybatis-3/zh/dynamic-sql.html)

简单说就是**不同的条件变化不同的sql语句；**

## 简介

- 动态SQL是MyBatis强大的特性之一，可以极大简化我们拼装sql的操作；
- 动态SQL元素和使用JSTL或其他类似基于XML的文本处理器相似；
- MyBatis采用功能强大的基于OGNL表达式来简化操作
  - if
  - choose(when,otherwise)
  - trim(where,set)
  - foreach

## 环境搭建

dbconfig.properties

```properties
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://Xxx:3306/mybatis
jdbc.username=root
jdbc.password=root
```

log4.properties

```properties
# Global logging configuration
log4j.rootLogger=trace, stdout
# MyBatis logging configuration...
log4j.logger=TRACE
# Console output...
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%5p [%t] - %m%n
```

mybatis-config.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    


    <properties resource="dbconfig.properties"></properties>
    
    <settings>
        <!--开启驼峰命名-->
        <setting name="mapUnderscoreToCamelCase" value="true"/>
        <setting name="lazyLoadingEnabled" value="true"/>
        <setting name="aggressiveLazyLoading" value="false"/>
    </settings>

    <typeAliases>
        <package name="top.tomxwd.mybatis.bean"/>
    </typeAliases>

    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="${jdbc.driver}"/>
                <property name="url" value="${jdbc.url}"/>
                <property name="username" value="${jdbc.username}"/>
                <property name="password" value="${jdbc.password}"/>
            </dataSource>
        </environment>
    </environments>

    <mappers>
        <package name="top.tomxwd.mybatis.mapper"/>
    </mappers>
</configuration>
```

pom.xml

```xml
<dependencies>
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
        <version>3.4.1</version>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>1.18.8</version>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>5.1.12</version>
    </dependency>
    <dependency>
        <groupId>log4j</groupId>
        <artifactId>log4j</artifactId>
        <version>1.2.17</version>
    </dependency>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
    </dependency>
</dependencies>

<build>
    <!--解决Intellij构建项目时，target/classes目录下不存在mapper.xml文件-->
    <resources>
        <resource>
            <directory>${basedir}/src/main/java</directory>
            <includes>
                <include>**/*.xml</include>
            </includes>
        </resource>
    </resources>
</build>
```

**注意，IDEA默认是不会吧mapper包下的xml文件扫描到的，所以要加build->resources；**

主要改成了包扫描，然后把mapper映射文件也放到mapper包了。

