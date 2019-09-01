---
title: 13_MyBatis_全局配置文件_databaseIdProvider_多数据库支持
date: 2019-08-09 18:55:51
tags: 
 - Java
 - Mybatis
categories:
 - Mybatis
---

# 13_MyBatis\_全局配置文件\_databaseIdProvider_多数据库支持

> [官网——databaseIdProvider](http://www.mybatis.org/mybatis-3/zh/configuration.html#databaseIdProvider)

- databaseIdProvider：支持多数据库厂商

  - type："DB_VENDOR"(这个也是个别名)

    作用就是得到数据库厂商的标识（驱动带来的getDatabaseProductName()），mybatis就能根据数据库厂商的标识来执行不同的sql

    - MySQL
    - Oracle
    - SQL Server

  - `<property>`

    - name：厂商标识
    - value：别名

在sql映射文件的标签上，用databaseId属性指定一个名称即可。

## mybatis-config.xml配置文件

还需要切换环境，这里我没有安装Oracle，不做调试。

```xml
<databaseIdProvider type="DB_VENDOR">
    <property name="MySQL" value="mysql"/>
    <property name="Oracle" value="oracle"/>
    <property name="SQL Server" value="sqlserver"/>
</databaseIdProvider>
```

## sql映射文件

```xml
<select id="getEmpById" resultType="EMP" databaseId="mysql">
    SELECT * FROM tbl_employee where id = #{id}
</select>
<select id="getEmpById" resultType="EMP" databaseId="oracle">
    SELECT EMPLOYEE_ID id,LAST_NAME lastName,EMAIL email FROM employees WHERE EMPLOYEE_ID=#{id}
</select>
```

