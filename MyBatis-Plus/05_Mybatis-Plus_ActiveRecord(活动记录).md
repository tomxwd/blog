---
title: 05_Mybatis-Plus_ActiveRecord(活动记录)
data: 2019-06-09 ‏‎‏‎‏‎13:13:00
tags: 
 - Java
 - Mybatis
 - Mybatis-Plus
categories:
 - Mybatis
 - Mybatis-Plus
---

# 05_Mybatis-Plus_ActiveRecord(活动记录)

Active Record(活动记录)，是一种**领域模型模式**，特点是一个模型类对应关系型数据库中的一个表，而模型类的一个实例对应表中的一条记录。
<br>
ActiveRecord一直广受动态语言（PHP、Ruby等）的喜爱，而Java作为准静态语言，对于ActiveRecord往往只能感叹其优雅，所以MP也在AR道路上进行了一定的探索。

----
## 如何使用AR模式
仅仅需要让实体类继承Model类并且实现主键指定方法即可。
<br>
相关代码：
```
public class Employee extends Model<Employee> {
	/**
	 * 该方法是Model的抽象方法 必须要实现
	 * 这里用来指定当前实体类的主键属性
	 */
	@Override
	protected Serializable pkVal() {
		return this.id;
	}
}
```
1. 继承Model
2. 实现pkVal，返回主键

---
## AR基本的CRUD操作
#### 插入操作
###### public boolean insert()
相关代码：
```
/**
 * AR插入操作
 * @throws Exception
 */
@Test
public void testARInsert() throws Exception {
  Employee emp = new Employee();
  emp.setLastName("Tommy");
  emp.setEmail("tommy@tomxwd.top");
  emp.setAge(35);
  boolean result = emp.insert();
  System.out.println("result: "+result);
}
```
日志输出：
```
DEBUG 06-09 13:36:01,570 ==>  Preparing: INSERT INTO tbl_employee ( last_name, email, age ) VALUES ( ?, ?, ? )  (JakartaCommonsLoggingImpl.java:54)
DEBUG 06-09 13:36:01,618 ==> Parameters: Tommy(String), tommy@tomxwd.top(String), 35(Integer) (JakartaCommonsLoggingImpl.java:54)
DEBUG 06-09 13:36:01,623 <==    Updates: 1 (JakartaCommonsLoggingImpl.java:54)
```
控制台输出：
```
result: true
```

---
#### 修改操作
###### public boolean updateById()
相关代码：
```
/**
 * AR修改操作
 * @throws Exception
 */
@Test
public void testARUpdate() throws Exception {
  Employee emp = new Employee();
  emp.setAge(15);
  emp.setId(16);
  boolean result = emp.updateById();
  System.out.println("result: "+result);
}
```
日志输出：
```
DEBUG 06-09 13:40:05,772 ==>  Preparing: UPDATE tbl_employee SET age=? WHERE id=?  (JakartaCommonsLoggingImpl.java:54)
DEBUG 06-09 13:40:05,830 ==> Parameters: 15(Integer), 16(Integer) (JakartaCommonsLoggingImpl.java:54)
DEBUG 06-09 13:40:05,832 <==    Updates: 1 (JakartaCommonsLoggingImpl.java:54)
```
控制台输出：
```
result: true
```

---
#### 查询操作
###### public T selectById()
相关代码：
```
/**
 * AR查询，根据id
 * @throws Exception
 */
@Test
public void testARSelectById() throws Exception {
  Employee emp = new Employee();
  emp.setId(16);
  Employee employee = emp.selectById();
  System.out.println(employee);
}
```
日志输出：
```
DEBUG 06-09 13:42:30,910 ==>  Preparing: SELECT id,last_name AS lastName,email,gender,age FROM tbl_employee WHERE id=?  (JakartaCommonsLoggingImpl.java:54)
DEBUG 06-09 13:42:30,964 ==> Parameters: 16(Integer) (JakartaCommonsLoggingImpl.java:54)
DEBUG 06-09 13:42:30,984 <==      Total: 1 (JakartaCommonsLoggingImpl.java:54)
```
控制台输出：
```
Employee [id=16, lastName=Tommy, email=tommy@tomxwd.top, gender=1, age=15]
```

###### public T selectById(Serializable id)
相关代码：
```
/**
 * AR查询，根据id,直接传入id值
 * @throws Exception
 */
@Test
public void testARSelectByIdSerializable() throws Exception {
  Employee emp = new Employee();
  Employee employee = emp.selectById(16);
  System.out.println(employee);
}
```
日志输出：
```
DEBUG 06-09 13:44:32,941 ==>  Preparing: SELECT id,last_name AS lastName,email,gender,age FROM tbl_employee WHERE id=?  (JakartaCommonsLoggingImpl.java:54)
DEBUG 06-09 13:44:32,979 ==> Parameters: 16(Integer) (JakartaCommonsLoggingImpl.java:54)
DEBUG 06-09 13:44:32,996 <==      Total: 1 (JakartaCommonsLoggingImpl.java:54)
```
控制台输出：
```
Employee [id=16, lastName=Tommy, email=tommy@tomxwd.top, gender=1, age=15]

```

###### public List<T> selectAll()
相关代码：
```
/**
 * AR查询，查询所有记录ALL
 * @throws Exception
 */
@Test
public void testARSelectAll() throws Exception {
  Employee emp = new Employee();
  List<Employee> emps = emp.selectAll();
  System.out.println(emps);
}
```
日志输出：
```
DEBUG 06-09 13:46:25,544 ==>  Preparing: SELECT id,last_name AS lastName,email,gender,age FROM tbl_employee  (JakartaCommonsLoggingImpl.java:54)
DEBUG 06-09 13:46:25,598 ==> Parameters:  (JakartaCommonsLoggingImpl.java:54)
DEBUG 06-09 13:46:25,626 <==      Total: 9 (JakartaCommonsLoggingImpl.java:54)
```
控制台输出：
```
[
  Employee [id=1, lastName=Tom, email=tom@tomxwd.top, gender=1, age=22],
  Employee [id=2, lastName=Jerry, email=jerry@tomxwd.top, gender=0, age=25],
  Employee [id=3, lastName=Black, email=black@tomxwd.top, gender=1, age=30],
  Employee [id=4, lastName=White, email=white@tomxwd.top, gender=0, age=35],
  Employee [id=8, lastName=Ken, email=ken@tomxwd.top, gender=1, age=30],
  Employee [id=12, lastName=Ken, email=ken@tomxwd.top, gender=1, age=30],
  Employee [id=13, lastName=Marry, email=marry@tomxwd.top, gender=0, age=15],
  Employee [id=15, lastName=1老师, email=laoshi@tomxwd.top, gender=0, age=20],
  Employee [id=16, lastName=Tommy, email=tommy@tomxwd.top, gender=1, age=15]
]
```

###### public List<T> selectList(Wrapper wrapper)
相关代码：
```
/**
 * AR查询，selectList，可以传入查询对象Wrapper
 * @throws Exception
 */
@Test
public void testARSelectList() throws Exception {
  Employee emp = new Employee();
  List<Employee> emps = emp.selectList(new EntityWrapper<Employee>().like("last_name", "Ken"));
  System.out.println(emps);
}
```
日志输出：
```
DEBUG 06-09 13:49:32,255 ==>  Preparing: SELECT id,last_name AS lastName,email,gender,age FROM tbl_employee WHERE (last_name LIKE ?)  (JakartaCommonsLoggingImpl.java:54)
DEBUG 06-09 13:49:32,322 ==> Parameters: %Ken%(String) (JakartaCommonsLoggingImpl.java:54)
DEBUG 06-09 13:49:32,357 <==      Total: 2 (JakartaCommonsLoggingImpl.java:54)
```
控制台输出：
```
[
  Employee [id=8, lastName=Ken, email=ken@tomxwd.top, gender=1, age=30],
  Employee [id=12, lastName=Ken, email=ken@tomxwd.top, gender=1, age=30]
]
```

###### public int selectCount(Wrapper wrapper)
相关代码：
```
/**
 * AR查询：selectCount
 * @throws Exception
 */
@Test
public void testARSelectCount() throws Exception {
  Employee emp = new Employee();
  int count = emp.selectCount(new EntityWrapper<Employee>().like("last_name", "K"));
  System.out.println("count: "+count);
}
```
日志输出：
```
DEBUG 06-09 13:51:55,245 ==>  Preparing: SELECT COUNT(1) FROM tbl_employee WHERE (last_name LIKE ?)  (JakartaCommonsLoggingImpl.java:54)
DEBUG 06-09 13:51:55,313 ==> Parameters: %K%(String) (JakartaCommonsLoggingImpl.java:54)
DEBUG 06-09 13:51:55,332 <==      Total: 1 (JakartaCommonsLoggingImpl.java:54)
```
控制台输出：
```
count: 3
```

---
#### 删除操作
***注意，当删除不存在的数据时，也会返回true。***
###### public boolean deleteById()
相关代码：
```
/**
 * AR删除：根据id删除——从对象setId
 * @throws Exception
 */
@Test
public void testARDeleteById() throws Exception {
  Employee emp = new Employee();
  emp.setId(15);
  boolean result = emp.deleteById();
  System.out.println("result: "+result);
}
```
日志输出：
```
DEBUG 06-09 14:07:16,965 ==>  Preparing: DELETE FROM tbl_employee WHERE id=?  (JakartaCommonsLoggingImpl.java:54)
DEBUG 06-09 14:07:17,010 ==> Parameters: 15(Integer) (JakartaCommonsLoggingImpl.java:54)
DEBUG 06-09 14:07:17,013 <==    Updates: 1 (JakartaCommonsLoggingImpl.java:54)
```
控制台输出：
```
result: true
```

###### public boolean deleteById(Serializable id)
相关代码：
```
/**
 * AR删除：根据id删除——直接传入id值
 * @throws Exception
 */
@Test
public void testARDeleteByIdSerializable() throws Exception {
  Employee emp = new Employee();
  boolean result = emp.deleteById(13);
  System.out.println("result: "+result);
}
```
日志输出：
```
DEBUG 06-09 14:08:43,546 ==>  Preparing: DELETE FROM tbl_employee WHERE id=?  (JakartaCommonsLoggingImpl.java:54)
DEBUG 06-09 14:08:43,594 ==> Parameters: 13(Integer) (JakartaCommonsLoggingImpl.java:54)
DEBUG 06-09 14:08:43,596 <==    Updates: 1 (JakartaCommonsLoggingImpl.java:54)
```
控制台输出：
```
result: true
```

###### public boolean delete(Wrapper wrapper)
相关代码：
```
/**
 * AR删除：根据条件删除
 * @throws Exception
 */
@Test
public void testARDeleteWrapper() throws Exception {
  Employee emp = new Employee();
  boolean result = emp.delete(new EntityWrapper<Employee>().like("age", "15"));
  System.out.println("result: "+result);
}
```
日志输出：
```
DEBUG 06-09 14:09:19,046 ==>  Preparing: DELETE FROM tbl_employee WHERE (age LIKE ?)  (JakartaCommonsLoggingImpl.java:54)
DEBUG 06-09 14:09:19,124 ==> Parameters: %15%(String) (JakartaCommonsLoggingImpl.java:54)
DEBUG 06-09 14:09:19,126 <==    Updates: 1 (JakartaCommonsLoggingImpl.java:54)
```
控制台输出：
```
result: true
```

---
#### 分页复杂操作
这里会返回Page对象，需要getRecords()才能得到具体数据，其他是分页结果等信息。
###### public Page<T> selectPage(Page<T> page,String whereClause, Object...args).getRecords();
相关代码：
```
/**
 * AR分页复杂操作
 * @throws Exception
 */
@Test
public void testSelectPage() throws Exception {
  Employee emp = new Employee();
  Page<Employee> emps = emp.selectPage(new Page<Employee>(2, 1), new EntityWrapper<Employee>().eq("last_name", "Ken"));
  System.out.println(emps);
  System.out.println(emps.getRecords());
}
```
日志输出：
```
DEBUG 06-09 14:19:42,949 ==>  Preparing: SELECT id,last_name AS lastName,email,gender,age FROM tbl_employee WHERE (last_name = ?)  (JakartaCommonsLoggingImpl.java:54)
DEBUG 06-09 14:19:43,024 ==> Parameters: Ken(String) (JakartaCommonsLoggingImpl.java:54)
```
控制台输出：
```
Page:{ [Pagination { total=0 ,size=1 ,pages=0 ,current=2 }], records-size:1 }
[Employee [id=12, lastName=Ken, email=ken@tomxwd.top, gender=1, age=30]]

```

---
## AR总结
1. AR模式提供了一种更加便捷的方式实现了CRUD操作，其本质还是调用了Mybatis对应的方法（sqlSession调用方法），类似于语法糖。
  - 语法糖：计算机学专家发明的一种术语。指**计算机语言中添加的某种语法，这种语法对原本语言的功能并没有影响，可以更方便开发者使用，可以避免出错的机会，让程序可读性更好。**
2. 到此，我们简单领略了Mybatis-Plus的魅力与高效率，值得一提的是：我们提供了强大的代码生成器，可以快速生成各类代码，真正做到了**即开即用**。
