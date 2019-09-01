---
title: 08_MyBatis_全局配置文件_settings_运行时行为设置
date: 2019-08-09 15:42:23
tags: 
 - Java
 - Mybatis
categories:
 - Mybatis
---

# 08_MyBatis\_全局配置文件\_settings_运行时行为设置

> [settings官方文档](http://www.mybatis.org/mybatis-3/zh/configuration.html#settings)

自个上官网看吧。

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
    
    <settings>
        <!--开启驼峰命名-->
        <setting name="mapUnderscoreToCamelCase" value="true"/>
    </settings>

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

这里仅关注settings标签的内容，开启了驼峰命名规则，而且需要注意的是这些标签的先后顺序，如果错了，会有提示，比如settings是要在properties之下的。