---
title: 14_JDBC_获取插入记录的主键值
date: 2019-08-23 09:28:11
tags: 
 - Java
categories:
 - JDBC
---

# 14_JDBC_获取插入记录的主键值

![取得数据库自动生成的主键](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/JDBC/14%E5%8F%96%E5%BE%97%E6%95%B0%E6%8D%AE%E5%BA%93%E8%87%AA%E5%8A%A8%E7%94%9F%E6%88%90%E7%9A%84%E4%B8%BB%E9%94%AE.png)

测试类：

```java
@Test
public void testGetKeyValue(){
    Connection conn = null;
    PreparedStatement ps = null;
    ResultSet rs = null;
    try {
        conn = JDBCTools.getConnection();
        String sql = "insert into customers (name,email,birth) values(?,?,?)";
        //ps = conn.prepareStatement(sql);
        // 改用重载的方法prepareStatement(sql,flag)来生成PreparedStatement
        ps = conn.prepareStatement(sql,Statement.RETURN_GENERATED_KEYS);
        ps.setObject(1,"ABCDE");
        ps.setObject(2,"abcde@qq.com");
        ps.setObject(3,new Date(new java.util.Date().getTime()));
        ps.executeUpdate();
        // 通过getGeneratedKeys()方法获取ResultSet对象
        rs = ps.getGeneratedKeys();
        if(rs.next()){
            Object object = rs.getObject(1);
            System.out.println("object = " + object);
        }
        ResultSetMetaData metaData = rs.getMetaData();
        for (int i = 0; i < metaData.getColumnCount(); i++) {
            System.out.println("metaData.getColumnName() = " + metaData.getColumnName(i + 1));
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



