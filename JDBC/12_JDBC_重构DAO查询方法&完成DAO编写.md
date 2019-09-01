---
title: 12_JDBC_重构DAO查询方法&完成DAO编写
date: 2019-08-23 07:54:02
tags: 
 - Java
categories:
 - JDBC
---

# 12_JDBC_重构DAO查询方法&完成DAO编写

主要完善查询一条、查询多条、通用更新、查一个值的情况。

BaseDao代码：

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
        T object = null;
        List<T> list = this.getForList(clazz, sql, args);
        if(list.size()>0){
            object = list.get(0);
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

    /**
     * 返回某条记录的某一个字段的值或者一个统计的值（一共有多少条记录等）
     * @param sql
     * @param args
     * @param <E>
     * @return
     */
    public <E> E getForValue(String sql,Object ...args){
        // 1. 得到结果集：应该只有一行且只有一列
        Connection conn = null;
        PreparedStatement ps = null;
        ResultSet rs = null;
        E obj = null;
        try {
            conn = JDBCTools.getConnection();
            ps = conn.prepareStatement(sql);
            for (int i = 0; i < args.length; i++) {
                ps.setObject(i+1,args[i]);
            }
            rs = ps.executeQuery();
            if(rs.next()){
                // 2.2 取得结果
                obj = (E) rs.getObject(1);
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
        // 2.2 取得结果
        return obj;
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

    @Test
    public void testBeanUtils() throws InvocationTargetException, IllegalAccessException {
        Object obj = new Student();
        System.out.println(obj);
        BeanUtils.setProperty(obj,"idCard","123");
        System.out.println("obj = " + obj);
    }

    @Test
    public void testGetForValue(){
        String sql = "select count(*) from customers";
        long i = getForValue(sql);
        System.out.println("i = " + i);
        
        sql = "select birth from customers where id=?";
        Date a = getForValue(sql,2);
        System.out.println("a = " + a);
        
        sql = "select max(grade) from examstudent";
        int m = getForValue(sql);
        System.out.println("m = " + m);
    }

}
```

