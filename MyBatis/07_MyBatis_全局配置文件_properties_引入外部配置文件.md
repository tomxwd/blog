---
title: 07_MyBatis_全局配置文件_properties_引入外部配置文件
date: 2019-08-09 15:42:23
tags: 
 - Java
 - Mybatis
categories:
 - Mybatis
---

# 07_MyBatis\_全局配置文件\_properties_引入外部配置文件

> [官方文档](http://www.mybatis.org/mybatis-3/zh/configuration.html#)

dbconfig.properties:

```properties
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://tomxwd.top:12389/mybatis
jdbc.username=root
jdbc.password=root
```

mybatis-config.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>

    <!--
        1. mybatis可以使用properties标签来引入外部properties配置文件的内容；
            resource:引入类路径下的资源
            url:引入网络路径或磁盘路径下的资源
    -->
    <properties resource="dbconfig.properties"></properties>

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
    <!--将我们写好的sql映射文件注册到全局配置文件中来-->
    <mappers>
        <mapper resource="EmployeeMapper.xml"/>
    </mappers>
</configuration>
```

虽然mybatis提供了这种形式，但是其实如果整合了spring，那么我们会使用spring来注入数据源，因此用不上这个标签。