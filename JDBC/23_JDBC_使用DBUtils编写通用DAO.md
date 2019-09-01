---
title: 23_JDBC_使用DBUtils编写通用DAO
date: 2019-08-26 08:06:21
tags: 
 - Java
categories:
 - JDBC
---

# 23_JDBC_使用DBUtils编写通用DAO

![Dao继承关系](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/JDBC/23Dao%E7%BB%A7%E6%89%BF%E5%85%B3%E7%B3%BB.png)

编写DAO接口：

```java
/**
 * 访问数据的接口
 * 里面定义好访问数据表的各种方法
 * @param <T>:DAO处理实体类的类型
 */
public interface DAO<T> {

    /**
     * 批量处理的方法
     * @param conn
     * @param sql
     * @param args
     * @return
     */
    Integer batch(Connection conn,String sql,Object ...args);

    /**
     * 返回具体的一个值，例如总人数，平均工资，某一个人的email等
     * @param conn
     * @param sql
     * @param args
     * @param <E>
     * @return
     */
    <E> E getForValue(Connection conn,String sql, Object ...args);

    /**
     * 返回T的一个集合
     * @param conn
     * @param sql
     * @param args
     * @return
     */
    List<T> getForList(Connection conn, String sql, Object ...args);

    /**
     * 返回一个T的对象
     * @param conn
     * @param sql
     * @param args
     * @return
     */
    T get(Connection conn,String sql,Object ...args) throws SQLException;

    /**
     * insert,update,delte
     * @param conn:数据库连接
     * @param sql：SQL语句
     * @param args：填充占位符的可变参数
     * @return
     */
    int update(Connection conn,String sql,Object ...args);


}
```

编写DAO的实现类JDBCDaoImpl：

```java
/**
 * 使用QueryRunner提供其具体的实现
 * @param <T> 子类需要传入的泛型类型
 */
public class JDBCDaoImpl<T> implements DAO<T> {

    private QueryRunner queryRunner = null;
    private Class<T> type;

    public JDBCDaoImpl(){
        queryRunner = new QueryRunner();
        Type genericSuperclass = getClass().getGenericSuperclass();
        Type[] actualTypeArguments = ((ParameterizedType) genericSuperclass).getActualTypeArguments();
        type = (Class) actualTypeArguments[0];
    }

    public Integer batch(Connection conn, String sql, Object... args) {
        return null;
    }

    public <E> E getForValue(Connection conn, String sql, Object... args) {
        return null;
    }

    public List<T> getForList(Connection conn, String sql, Object... args) {
        return null;
    }

    public T get(Connection conn, String sql, Object... args) throws SQLException {
        return queryRunner.query(conn,sql,new BeanHandler<T>(type),args);
    }

    public int update(Connection conn, String sql, Object... args) {
        return 0;
    }
}
```

编写CustomerDao继承JDBCDaoImpl：

```java
public class CustomerDao extends JDBCDaoImpl<Customer> {
}
```

测试get方法：

```java
public class CustomerDaoTest {

    CustomerDao dao = new CustomerDao();

    @Test
    public void testGet(){
        Connection conn = null;
        try {
            conn = JDBCTools.getConnection();
            String sql = "select id,name,email,birth from customers where id=?";
            Customer customer = dao.get(conn, sql, 2);
            System.out.println("customer = " + customer);
        } catch (IOException e) {
            e.printStackTrace();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (SQLException e) {
            e.printStackTrace();
        } finally {
            JDBCTools.release(null,conn);
        }
    }

}
```

输出：

```
customer = Customer{id=2, name='tom', email='abc@qq.com', birth=2990-12-12}
```





