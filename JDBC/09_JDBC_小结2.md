---
title: 09_JDBC_小结2
date: 2019-08-22 15:12:21
tags: 
 - Java
categories:
 - JDBC
---

# 09_JDBC_小结2

## PreparedStatement

1. why
   - 使用Statement需要进行拼写SQL语句，加单引号之类的，很辛苦而且容易出错。
   - 使用Statement可能会发生sql注入。
     - SQL注入是利用某些系统没有对用户输入的数据进行充分检查，而在用户输入的数据中注入非法的SQL语句或片段，从而利用系统的SQL引擎完成恶意行为的做法，对Java而言只需要用PreparedStatement替代Statement即可防止。
2. what
   - 是Statement的子接口
   - 可以传入带占位符的SQL语句，并且提供了补充占位符变量的方法。
3. how
   - 创建PreparedStatement，用connection.preparedStatement(sql)并且要传入sql。
   - 调用preparedStatement的setXxx(int index,Object val)设置占位符的值。**index从1开始。**
   - 执行SQL语句：executeQuery()或者executeUpdate()，注意这时候不用传入sql了。



## ResultSetMetaData

1. why

   - 如果只有一个结果集，但不知道该结果集中有多少列，列的名字是什么。

   - 编写通用的查询方法的时候需要使用

     ```java
     public <T> T get(Class<T> clazz,String sql,Object ...args)
     ```

2. what

   - 用于描述ResultSet的对象。

3. how

   - 得到ResultSetMetaData对象：调用ResultSet的getMetaData()方法
   - ResultSetMetaData有哪些好用的方法：
     - int getColumnCount()：结果集中的列数
     - String getColumnLabel(int index)：获取指定的列的**别名**，其中索引从1开始。

   ```java
   for(int i=0;i<rsmd.getColumnCount();i++){
       String columnLabel = rsmd.getColumnLabel(i+1);
   }
   ```





## 通用查询

### 单条记录

```java
public static <T> T get(Class clazz,String sql,Object ...args){
    T entity = null;
    Connection connection = null;
    PreparedStatement ps = null;
    ResultSet resultSet = null;
    try {
        connection = JDBCTools.getConnection();
        ps = connection.prepareStatement(sql);
        for (int i = 0; i < args.length; i++) {
            ps.setObject(i+1,args[i]);
        }
        resultSet = ps.executeQuery();
        ResultSetMetaData metaData = resultSet.getMetaData();
        if(resultSet.next()){
            // 反射创建对象
            entity = (T) clazz.newInstance();
            // 通过解析sql语句来判断到底选择了哪些列，以及需要被entity对象的哪些属性赋值
            for (int i = 0; i < metaData.getColumnCount(); i++) {
                BeanUtils.setProperty(entity,metaData.getColumnLabel(i+1),resultSet.getObject(metaData.getColumnLabel(i+1)));
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
        JDBCTools.release(resultSet,ps,connection);
    }
    return entity;
}
```

### 多条记录

```java
public static <T> List<T> getForList(Class<T> clazz,String sql,Object ...args){
    List<T> list = new ArrayList<T>();
    Connection connection = null;
    PreparedStatement ps = null;
    ResultSet resultSet = null;
    try {
        connection = getConnection();
        ps = connection.prepareStatement(sql);
        for (int i = 0; i < args.length; i++) {
            ps.setObject(i+1,args[i]);
        }
        resultSet = ps.executeQuery();
        ResultSetMetaData metaData = resultSet.getMetaData();
        while (resultSet.next()){
            T object = clazz.newInstance();
            for (int i = 0; i < metaData.getColumnCount(); i++) {
                BeanUtils.setProperty(object,metaData.getColumnLabel(i+1),resultSet.getObject(i+1));
            }
            list.add(object);
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
        release(resultSet,ps,connection);
    }
    return list;
}
```

主要用到反射、JDBC、JDBC元数据的知识。

