---
title: 21_JDBC_使用DBUtils进行更新操作
date: 2019-08-23 16:44:44
tags: 
 - Java
categories:
 - JDBC
---

# 21_JDBC_使用DBUtils进行更新操作

## Apache-DBUtils简介

commons-dbutils是Apache组织提供的一个开源JDBC工具类库，它是对JDBC的简单封装，学习成本极低，并且使用dbutils能极大简化jdbc编码的工作量，同时也不会影响程序的性能。

### API介绍

- org.apache.commons.dbutils.QueryRunner
- org.apache.commons.dbutils.ResultSetHandler
- 工具类
  - org.apache.commons.dbutils.DbUtils



## 测试

导入依赖：

```xml
<dependency>
    <groupId>commons-dbutils</groupId>
    <artifactId>commons-dbutils</artifactId>
    <version>1.7</version>
</dependency>
```

测试方法：

```java
/**
 * 测试QueryRunner类的update方法
 */
@Test
public void testQueryRunnerUpdate(){
    Connection conn = null;
    // 1. 创建QueryRunner的实现类
    QueryRunner queryRunner = new QueryRunner();
    // 2. 使用其update方法
    String sql ="delete from customers where id in(?,?)";
    try {
        conn = JDBCTools.getConnection();
        queryRunner.update(conn,sql,12,11);
    } catch (IOException e) {
        e.printStackTrace();
    } catch (ClassNotFoundException e) {
        e.printStackTrace();
    } catch (SQLException e) {
        e.printStackTrace();
    } finally {
        JDBCTools.release(null,null,conn);
    }
}
```



