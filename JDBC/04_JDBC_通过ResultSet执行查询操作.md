---
title: 04_JDBC_通过ResultSet执行查询操作
date: 2019-08-21 16:31:36
tags: 
 - Java
categories:
 - JDBC
---

# 04_JDBC_通过ResultSet执行查询操作

## ResultSet结果集

- 通过调用Statement对象的excuteQuery()方法创建该对象
- ResultSet对象以逻辑表格的形式封装了执行数据库操作的结果集，ResultSet接口由数据库厂商实现；
- ResultSet对象维护了一个指向当前数据行的游标，初始的时候，游标在第一行之前，可以通过ResultSet对象的next()方法移动到下一行；
- ResultSet接口的常用方法：
  - boolean next()
  - getString()
  - ......



![ResultSet图示](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/JDBC/04ResultSet%E5%9B%BE%E7%A4%BA.png)

![数据类型转换表](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/JDBC/04%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B%E8%BD%AC%E6%8D%A2%E8%A1%A8.png)

ResultSet:结果集，封装了使用JDBC进行查询的结果

1. 调用Statement对象的executeQuery(sql)可以得到结果集

2. ResultSet返回的实际上就是一张数据表，有一个指针指向数据表的第一行的前面，可以调用next方法检测下一行是否有效，若有效，该方法返回true，且指针下移。相当于Iterator对象的hasNext()和next()方法的结合体。

3. 当指针定位到一行的时候，可以通过调用getXxx(index)或getXxx(columnName)方法来获取每一列的值。

   例如：

   - getInt(1)
   - getString("name")

4. ResultSet当然也需要关闭



准备工作：

1. JDBCTool工具类写多一个重载的释放资源方法。

   ```java
   /**
    * 关闭ResultSet、Statement和Connection
    * @param statement
    * @param connection
    */
   public static void release(ResultSet resultSet,Statement statement, Connection connection){
       if(resultSet!=null){
           try {
               resultSet.close();
           } catch (Exception e){
               e.printStackTrace();
           }
       }
       release(statement,connection);
   }
   ```

   

2. 编写代码测试单条记录查询：

   代码：

   ```java
   /**
    * ResultSet:结果集，封装了使用JDBC进行查询的结果
    * 1. 调用Statement对象的executeQuery(sql)可以得到结果集
    * 2. ResultSet返回的实际上就是一张数据表，有一个指针指向数据表的第一行的前面，
    *    可以调用next方法检测下一行是否有效，若有效，该方法返回true，且指针下移。
    *    相当于Iterator对象的hasNext()和next()方法的结合体
    * 3. 当指针定位到一行的时候，可以通过调用getXxx(index)或getXxx(columnName)
    *    方法来获取每一列的值。
    *    例如：
    *    getInt(1)
    *    getString("name")
    * 4. ResultSet当然也需要关闭
    */
   @Test
   public void testResultSet(){
       // 获取id=2的customers数据表的记录并打印
       Connection conn = null;
       Statement statement = null;
       ResultSet resultSet = null;
       try {
           // 1. 获取Connection
           conn = JDBCTool.getConnection();
           // 2. 获取Statement
           statement = conn.createStatement();
           // 3. 准备SQL
           String sql = "SELECT ID,NAME,EMAIL,BIRTH FROM customers WHERE ID=2";
           // 4. 执行查询，得到ResultSet
           resultSet = statement.executeQuery(sql);
           // 5. 处理ResultSet
           if(resultSet.next()){
               int id = resultSet.getInt(1);
               String name = resultSet.getString("name");
               String email = resultSet.getString(3);
               Date brith = resultSet.getDate(4);
               System.out.println("id = " + id);
               System.out.println("name = " + name);
               System.out.println("email = " + email);
               System.out.println("brith = " + brith);
           }
       } catch (Exception e) {
           e.printStackTrace();
       } finally {
           // 6. 关闭数据库资源
           JDBCTool.release(resultSet,statement,conn);
       }
   }
   ```

   输出：

   ```java
   id = 2
   name = tom
   email = abc@qq.com
   brith = 2990-12-12
   ```

   

3. 编写代码测试多条记录查询：

   只需要把单条查询的测试代码中的SQL语句WHERE条件删除，以及把处理ResultSet的if改成while即可。

   代码：

   ```java
   @Test
   public void testResultSet(){
       // 获取id=2的customers数据表的记录并打印
       Connection conn = null;
       Statement statement = null;
       ResultSet resultSet = null;
       try {
           // 1. 获取Connection
           conn = JDBCTool.getConnection();
           // 2. 获取Statement
           statement = conn.createStatement();
           // 3. 准备SQL
           String sql = "SELECT ID,NAME,EMAIL,BIRTH FROM customers";
           // 4. 执行查询，得到ResultSet
           resultSet = statement.executeQuery(sql);
           // 5. 处理ResultSet
           while (resultSet.next()){
               System.out.println("-------------------");
               int id = resultSet.getInt(1);
               String name = resultSet.getString("name");
               String email = resultSet.getString(3);
               Date brith = resultSet.getDate(4);
               System.out.println("id = " + id);
               System.out.println("name = " + name);
               System.out.println("email = " + email);
               System.out.println("brith = " + brith);
           }
       } catch (Exception e) {
           e.printStackTrace();
       } finally {
           // 6. 关闭数据库资源
           JDBCTool.release(resultSet,statement,conn);
       }
   }
   ```

   输出：

   ```
   ----------
   id = 2
   name = tom
   email = abc@qq.com
   brith = 2990-12-12
   ----------
   id = 3
   name = AXX!D
   email = !!@bc@qq.com
   brith = 2900-12-12
   ----------
   id = 4
   name = XX
   email = 123@qq.com
   brith = 2019-08-05
   ```





