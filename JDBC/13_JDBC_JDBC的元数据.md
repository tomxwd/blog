---
title: 13_JDBC_JDBC的元数据
date: 2019-08-23 08:57:51
tags: 
 - Java
categories:
 - JDBC
---

# 13_JDBC_JDBC的元数据

## 测试两个类

### DatabaseMetaData：

```java
/**
 * DatabaseMetaData是描述数据库的元数据对象
 * 可以由Connection取得
 */
@Test
public void testDatabaseMetaData(){
    Connection conn = null;
    ResultSet rs = null;
    try {
        conn = JDBCTools.getConnection();
        // 可以得到数据库本身的一些基本信息
        DatabaseMetaData dbMetaData = conn.getMetaData();
        // 得到数据库的版本号
        System.out.println("dbMetaData.getDatabaseMajorVersion() = " + dbMetaData.getDatabaseMajorVersion());
        // 得到数据库连接的用户名
        System.out.println("dbMetaData.getUserName() = " + dbMetaData.getUserName());
        // 数据库连接url
        System.out.println("dbMetaData.getURL() = " + dbMetaData.getURL());
        // 得到MySql中有哪些数据库
        rs = dbMetaData.getCatalogs();
        while (rs.next()){
            Object object = rs.getObject(1);
            System.out.println("object = " + object);
        }
    } catch (IOException e) {
        e.printStackTrace();
    } catch (ClassNotFoundException e) {
        e.printStackTrace();
    } catch (SQLException e) {
        e.printStackTrace();
    } finally {
        JDBCTools.release(rs,null,conn);
    }
}
```

### ResultSetMetaData：

```java
/**
  * ResultSetMetaData：描述结果集的元数据
  * 可以得到结果集中的基本信息：结果集中有哪些列，列名，列的别名等。
  */
@Test
public void testResultSetMetaData(){
    Connection conn = null;
    PreparedStatement ps = null;
    ResultSet rs = null;
    try {
        conn = JDBCTools.getConnection();
        String sql = "select id,name,email,birth from customers";
        ps = conn.prepareStatement(sql);
        rs = ps.executeQuery();
        // 获取ResultSetMetaData对象
        ResultSetMetaData metaData = rs.getMetaData();
        // 得到列的个数
        int columnCount = metaData.getColumnCount();
        System.out.println("columnCount = " + columnCount);
        for (int i = 0; i < columnCount; i++) {
            // 得到列名
            System.out.println("别名("+(i+1)+") = " + metaData.getColumnName(i + 1));
            // 得到列的别名
            System.out.println("列名("+(i+1)+") = " + metaData.getColumnLabel(i + 1));
        }
    } catch (IOException e) {
        e.printStackTrace();
    } catch (ClassNotFoundException e) {
        e.printStackTrace();
    } catch (SQLException e) {
        e.printStackTrace();
    } finally {
        JDBCTools.release(rs,ps,conn);
    }
}
```

