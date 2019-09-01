---
title: 10_JDBC_DAO设计模式
date: 2019-08-22 15:41:36
tags: 
 - Java
categories:
 - JDBC
---

# 10_JDBC_DAO设计模式

DAO：Data Access Object

1. what：
   - 访问数据信息的类，包含了对数据的CRUD（create,retrieve,update,delete），而不包含任何业务相关的信息。
   - DAO可以被子类直接继承或者直接使用。
2. why：
   - 实现功能的模块化。
   - 更有利于代码的维护和升级。
3. how
   - 使用JDBC编写DAO可能会包含的方法：
     - `void update(String sql,Object ...args);`Insert,Update,Delete操作都可以包含在其中；
     - `<T> T get(Class<T> clazz,String sql,Object ...args);`返回单条记录；
     - `<T> List<T> getForList(Class<T> clazz,String sql,Object ...args);`返回多条记录；
     - `<E> E getForValue(String sql,Object ...args);`返回某条记录的某一个字段的值或者一个统计的值（一共有多少条记录）；

编写一个BaseDao类：

```java
public class BaseDao {

    public void update(String sql,Object ...args){
        Connection conn = null;
        PreparedStatement ps = null;
        try {
            conn = JDBCTools.getConnection();
            ps = conn.prepareStatement(sql);
            for (int i = 0; i < args.length; i++) {
                ps.setObject(i+1,args[i]);
            }
            ps.executeUpdate();
        } catch (IOException e) {
            e.printStackTrace();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (SQLException e) {
            e.printStackTrace();
        } finally {
            JDBCTools.release(ps,conn);
        }
    }

    public <T> T get(Class<T> clazz,String sql,Object ...args){
        Connection conn = null;
        PreparedStatement ps = null;
        ResultSet rs = null;
        T object = null;
        try {
            conn = JDBCTools.getConnection();
            ps = conn.prepareStatement(sql);
            for (int i = 0; i < args.length; i++) {
                ps.setObject(i+1,args[i]);
            }
            rs = ps.executeQuery();
            ResultSetMetaData metaData = rs.getMetaData();
            if(rs.next()){
                object = clazz.newInstance();
                for (int i = 0; i < metaData.getColumnCount(); i++) {
                    BeanUtils.setProperty(object,metaData.getColumnLabel(i+1),rs.getObject(i+1));
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (SQLException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (InstantiationException e) {
            e.printStackTrace();
        } catch (InvocationTargetException e) {
            e.printStackTrace();
        } finally {
            JDBCTools.release(rs,ps,conn);
        }
        return object;
    }

    public <T> List<T> getForList(Class<T> clazz,String sql,Object ...args){
        Connection conn = null;
        PreparedStatement ps = null;
        ResultSet rs = null;
        List<T> list = new ArrayList<T>();
        try {
            conn = JDBCTools.getConnection();
            ps = conn.prepareStatement(sql);
            for (int i = 0; i < args.length; i++) {
                ps.setObject(i+1,args[i]);
            }
            rs = ps.executeQuery();
            ResultSetMetaData metaData = rs.getMetaData();
            while (rs.next()){
                T t = clazz.newInstance();
                for (int i = 0; i < metaData.getColumnCount(); i++) {
                    BeanUtils.setProperty(t,metaData.getColumnLabel(i+1),rs.getObject(i+1));
                }
                list.add(t);
            }
        } catch (IOException e) {
            e.printStackTrace();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (SQLException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (InstantiationException e) {
            e.printStackTrace();
        } catch (InvocationTargetException e) {
            e.printStackTrace();
        } finally {
            JDBCTools.release(rs,ps,conn);
        }
        return list;
    }

    public <E> E getForValue(String sql,Object ...args){
        return null;
    }


}
```

测试：

```java
public class TestDao extends BaseDao {

    @Test
    public void testGet(){
        String sql = "select id,name,email,birth from customers where id=?";
        Customer customer = get(Customer.class, sql, 2);
        System.out.println("customer = " + customer);
    }

    @Test
    public void testGetForList(){
        String sql = "select id,name,email,birth from customers";
        List<Customer> list = getForList(Customer.class, sql);
        for (Customer customer : list) {
            System.out.println("customer = " + customer);
        }
    }

    @Test
    public void testUpdate(){
        String sql = "insert into customers (name,email,birth) values(?,?,?)";
        update(sql,"xxxxx","xxxxx@qq.com","1899-12-12");
    }

}
```

