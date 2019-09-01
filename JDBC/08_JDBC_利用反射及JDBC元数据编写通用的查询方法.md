---
title: 08_JDBC_利用反射及JDBC元数据编写通用的查询方法
date: 2019-08-22 13:50:13
tags: 
 - Java
categories:
 - JDBC
---

# 08_JDBC_利用反射及JDBC元数据编写通用的查询方法

## 反射规则以及步骤

![反射规则以及步骤](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/JDBC/08%E5%8F%8D%E5%B0%84%E8%A7%84%E5%88%99%E4%BB%A5%E5%8F%8A%E6%AD%A5%E9%AA%A4.png)





## 使用JDBC驱动程序处理元数据

第三步需要用到：

- Java通过JDBC获取连接后，得到一个Connection对象，可以从这个对象获取**有关数据库管理系统的各种信息**，包括数据库中的各个表，表中的各个列、数据类型、触发器、存储过程等各方面的信息。根据这些信息，JDBC可以访问一个事先并不了解的数据库。
- 获取这些信息的方法都是在**DatabaseMetaData**类的对象上实现的，而DatabaseMetaData对象是在Connection对象上获取的。



## DatabaseMetaData类

DatabaseMetaData类中提供了许多方法用于获得数据源的各种信息，通过这些方法可以非常详细的了解数据库的信息：

- getURL()：返回一个String类对象，代表数据库的URL；
- getUserName()：返回连接当前数据库管理系统的用户名；
- isReadOnly()：返回一个boolean值，指示数据库是否只允许读操作；
- getDatabaseProductName()：返回数据库的版本号；
- getDriverName()：返回驱动程序的名称；
- getDriverVersion()：返回程序驱动的版本号；



## ResultSetMetaData类

可用于获取关于ResultSet对象中列的类型和属性信息的对象：

- getColumnName(int column)：获取指定列的名称；
- getColumnCount()：返回当前ResultSet对象中的列数；
- getColumnTypeName()：检索指定列的数据库特定的类型名称；
- getColumnDisplaySize(int column)：指示指定列的最大标准宽度，以字符为单位；
- isNullable(int column)：指示指定列中的值是否可以为null；
- isAutoIncrement(int column)：指定是否自动为指定列进行编号，这样这些列仍然是只读的；



ResultSetMetaData

- what：是用于描述ResultSet的元数据对象，即从中可以获取到结果集中有多少列，列名是什么等信息。
- how：
  - 得到ResultSetMetaData对象：调用ResultSet的getMetaData()方法；
  - ResultSetMetaData有哪些好用的方法：
    - int getColumnCount()：SQL语句中包含哪些列，返回总数
    - String getColumnLabel(int column)：获取指定列的别名，索引从1开始



## 利用反射和元数据编写代码实现通用查询方法

![反射与元数据编写通用方法思路](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/JDBC/08%E5%8F%8D%E5%B0%84%E4%B8%8E%E5%85%83%E6%95%B0%E6%8D%AE%E7%BC%96%E5%86%99%E9%80%9A%E7%94%A8%E6%96%B9%E6%B3%95%E6%80%9D%E8%B7%AF.png)

要用到反射赋值给属性，这里用Apache的benautils包：

```xml
<dependency>
    <groupId>commons-beanutils</groupId>
    <artifactId>commons-beanutils</artifactId>
    <version>1.9.3</version>
</dependency>
```

测试大概的需求，先不抽方法：

```java
@Test
public void testResultSetMetaData() throws IllegalAccessException, InstantiationException {
    Connection connection = null;
    PreparedStatement ps = null;
    ResultSet resultSet = null;
    try {
        connection = JDBCTools.getConnection();
        String sql = "select flowid,type,idCard,examCard,studentName,location,grade from examstudent where flowid = ?";
        ps = connection.prepareStatement(sql);
        ps.setObject(1,10);
        resultSet = ps.executeQuery();

        Map<String,Object> values = new HashMap<String,Object>();

        while (resultSet.next()){
            // 1. 得到ResultSetMetaData对象
            ResultSetMetaData metaData = resultSet.getMetaData();
            // 2. 打印每一列的列名
            for (int i = 0; i < metaData.getColumnCount(); i++) {
                String columnLabel = metaData.getColumnLabel(i+1);
                Object columnValue = resultSet.getObject(columnLabel);
                values.put(columnLabel,columnValue);
            }
        }

        System.out.println("values = " + values);

        Class clazz = Student.class;
        Object object = clazz.newInstance();
        for(Map.Entry<String,Object> entry:values.entrySet()){
            String fieldName = entry.getKey();
            Object fieldValue = entry.getValue();
            BeanUtils.setProperty(object,fieldName,fieldValue);

        }
        System.out.println(object);
    } catch (IOException e) {
        e.printStackTrace();
    } catch (ClassNotFoundException e) {
        e.printStackTrace();
    } catch (SQLException e) {
        e.printStackTrace();
    } catch (InvocationTargetException e) {
        e.printStackTrace();
    } finally {
        JDBCTools.release(resultSet,ps,connection);
    }
}
```

这里其实也可以省略map的使用，直接给bean赋值。

抽取出来的通用代码：

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

测试上述方法：

```java
@Test
public void testGet(){
    String sql = "select id,name,email,birth from customers where id = ?";
    Customer customer = JDBCTools.get(Customer.class,sql,5);
    System.out.println("customer = " + customer);

    sql = "select flowid,type,idCard,examCard,studentName,location,grade from examstudent where flowid = ?";
    Student student = JDBCTools.get(Student.class,sql,10);
    System.out.println("student = " + student);
}
```

输出：

```
customer = Customer{id=5, name='ttt', email='1dzts@qq.com', birth=Fri Apr 25 00:00:00 CST 2014}
student = Student{flowId=0, type=0, idCard='12312332412', examCard='1333232', studentName='tom', location='深圳', grade=380}

```

