---
title: 04_Mybatis-Plus_条件构造器_EntityWrapper
data: 2019-06-09 ‏‎‏‎10:59:24
tags: 
 - Java
 - Mybatis
 - Mybatis-Plus
categories:
 - Mybatis
 - Mybatis-Plus
---

# 04_Mybatis-Plus_条件构造器_EntityWrapper

---

## EntityWrapper简介
1. Mybatis-Plus通过EntityWrapper（简称**EW**，MP封装的一个查询条件构造器）或者Condition（与EW类似）来让用户自由的构建查询条件，简单便捷，没有额外的负担，能够有效提高开发效率。
2. 实体包装器，主要用于处理SQL拼接、排序、实体参数查询等。
3. **注意：使用的是数据库的字段名，而不是java属性。**
4. 条件参数说明：

|查询方式|说明|
|-|-|
|setSqlSelect|设置SELECT查询字段|
|where|WHERE语句，拼接 - WHERE条件|
|and|AND语句，拼接 - AND 字段=值|
|andNew|AND语句，拼接 - AND （字段=值）|
|or|OR语句，拼接 - OR 字段=值|
|orNew|OR语句，拼接 - OR（字段=值）|
|eq|等于=|
|allEq|基于map内容等于=|
|ne|不等于<>|
|gt|大于>|
|ge|大于等于>=|
|lt|小于<|
|le|小于等于<=|
|like|模糊查询LIKE|
|notLike|模糊查询NOT LIKE|
|in|IN查询|
|notin|NOT IN查询|
|isNull|NULL值查询|
|isNotNull|IS NOT NULL|
|groupBy|分组GROUP BY|
|having|HAVING关键词|
|orderBy|排序ORDER BY|
|orderAsc|ASC排序ORDER BY|
|orderDesc|DESC排序ORDER BY|
|exists|EXISTS条件语句|
|notExists|NOT EXISTS条件语句|
|between|BETWEEN条件语句|
|notBetween|NOT BETWEEN条件语句|
|addFilter|自由拼接SQL|
|last|拼接在最后，例如last("LIMIT 1")|

## 简单使用
>需要分页查询tbl_employee表中，年龄在18-50，性别为男，姓名为Xxx的所有用户（Ken）

相关代码：
```
/**
 * 条件构造器EntityWrapper
 * @throws Exception
 */
@Test
public void testEntityWrapper() throws Exception {
  // 需要分页查询tbl_employee表中，年龄在18-50，性别为男，姓名为Xxx的所有用户（Ken）

  List<Employee> list = employeeMapper.selectPage(new Page<Employee>(1,3), new EntityWrapper<Employee>().between("age", 18, 50).eq("gender", 1).eq("last_name", "KEN"));
  System.out.println(list);

}
```
日志输出：
```
DEBUG 06-09 11:11:14,841 ==>  Preparing: SELECT id,last_name AS lastName,email,gender,age FROM tbl_employee WHERE (age BETWEEN ? AND ? AND gender = ? AND last_name = ?)  (JakartaCommonsLoggingImpl.java:54)
DEBUG 06-09 11:11:14,872 ==> Parameters: 18(Integer), 50(Integer), 1(Integer), KEN(String) (JakartaCommonsLoggingImpl.java:54)
DEBUG 06-09 11:11:14,891 <==      Total: 2 (JakartaCommonsLoggingImpl.java:54)
```
控制台输出：
```
[
  Employee [id=8, lastName=Ken, email=ken@tomxwd.top, gender=1, age=40],
  Employee [id=12, lastName=Ken, email=ken@tomxwd.top, gender=1, age=40]
]
```

---
## 带条件的查询
#### selectList(@Param("ew") Wrapper<T> wrapper);
相关代码：
```
/**
 * selectList(@Param("ew") Wrapper<T> wrapper);
 * @throws Exception
 */
@Test
public void testSelectList() throws Exception {
  // 查询tbl_employee表中，性别为女，姓名中带“老师” 或者 邮箱中带有a的
  List<Employee> emps = employeeMapper.selectList(
      new EntityWrapper<Employee>()
      .eq("gender", 0)
      .like("last_name", "老师")
//				.or()   //WHERE (gender = ? AND last_name LIKE ? OR email LIKE ?)
      .orNew()  //WHERE (gender = ? AND last_name LIKE ?) OR (email LIKE ?)
      .like("email", "a"));
  System.out.println(emps);
}
```
日志输出：
```
DEBUG 06-09 12:33:35,887 ==>  Preparing: SELECT id,last_name AS lastName,email,gender,age FROM tbl_employee WHERE (gender = ? AND last_name LIKE ?) OR (email LIKE ?)  (JakartaCommonsLoggingImpl.java:54)
DEBUG 06-09 12:33:35,932 ==> Parameters: 0(Integer), %老师%(String), %a%(String) (JakartaCommonsLoggingImpl.java:54)
DEBUG 06-09 12:33:35,953 <==      Total: 3 (JakartaCommonsLoggingImpl.java:54)
```
控制台输出：
```
[
  Employee [id=3, lastName=Black, email=black@tomxwd.top, gender=1, age=30],
  Employee [id=13, lastName=Marry, email=marry@tomxwd.top, gender=0, age=15],
  Employee [id=15, lastName=1老师, email=laoshi@tomxwd.top, gender=0, age=20]
]
```
需要注意：or()和orNew()的区别在于是否另起一个or。如下：
```
  or() : WHERE (gender = ? AND last_name LIKE ? OR email LIKE ?)
  orNew : WHERE (gender = ? AND last_name LIKE ?) OR (email LIKE ?)
```
---
## 带条件的修改
#### update(@Param("et") T  entity,@Param("ew") Wrapper<T> wrapper)
相关代码：
```
/**
 * 條件構造器的修改操作
 * 把名字为Ken的，性别改为女，年龄改为30岁
 * @throws Exception
 */
@Test
public void testUpdate() throws Exception {
  Employee emp = new Employee();
  emp.setAge(30);
  emp.setGender(0);
  Integer result = employeeMapper.update(emp, new EntityWrapper<Employee>().eq("last_name", "KEN"));
  System.out.println("result: "+result);
}
```
日志输出：
```
DEBUG 06-09 12:45:49,230 ==>  Preparing: UPDATE tbl_employee SET gender=?, age=? WHERE (last_name = ?)  (JakartaCommonsLoggingImpl.java:54)
DEBUG 06-09 12:45:49,282 ==> Parameters: 0(Integer), 30(Integer), KEN(String) (JakartaCommonsLoggingImpl.java:54)
DEBUG 06-09 12:45:49,284 <==    Updates: 3 (JakartaCommonsLoggingImpl.java:54)
```
控制台输出：
```
result: 3
```

---
## 带条件的删除
#### delete(@Param("ew") Wrapper<T> wrapper)
相关代码：
```
/**
 * 条件构造器的删除操作
 * 把名字为Ken且年龄为50的删除
 * @throws Exception
 */
@Test
public void testDelete() throws Exception {
  Integer result = employeeMapper.delete(new EntityWrapper<Employee>().eq("last_name", "KEN").eq("age", 50));
  System.out.println("result: "+result);
}
```
日志输出：
```
DEBUG 06-09 12:53:28,412 ==>  Preparing: DELETE FROM tbl_employee WHERE (last_name = ? AND age = ?)  (JakartaCommonsLoggingImpl.java:54)
DEBUG 06-09 12:53:28,473 ==> Parameters: KEN(String), 50(Integer) (JakartaCommonsLoggingImpl.java:54)
DEBUG 06-09 12:53:28,476 <==    Updates: 1 (JakartaCommonsLoggingImpl.java:54)
```
控制台输出：
```
result: 1
```

---
## 其他常用方法
相关代码：
```
/**
 * 查询性别为女，根据age排序（asc/desc），并进行简单分页。
 * @throws Exception
 */
@Test
public void testCommon() throws Exception {
  List<Employee> emps = employeeMapper.selectList(
      new EntityWrapper<Employee>()
      .eq("gender", 0)
      .orderDesc(Arrays.asList(new String[] {"age"})));
  System.out.println(emps);
}
```
日志输出：
```
DEBUG 06-09 12:58:38,939 ==>  Preparing: SELECT id,last_name AS lastName,email,gender,age FROM tbl_employee WHERE (gender = ?) ORDER BY age DESC  (JakartaCommonsLoggingImpl.java:54)
DEBUG 06-09 12:58:39,001 ==> Parameters: 0(Integer) (JakartaCommonsLoggingImpl.java:54)
DEBUG 06-09 12:58:39,030 <==      Total: 6 (JakartaCommonsLoggingImpl.java:54)
```
控制台输出：
```
[
  Employee [id=4, lastName=White, email=white@tomxwd.top, gender=0, age=35],
  Employee [id=8, lastName=Ken, email=ken@tomxwd.top, gender=0, age=30],
  Employee [id=12, lastName=Ken, email=ken@tomxwd.top, gender=0, age=30],
  Employee [id=2, lastName=Jerry, email=jerry@tomxwd.top, gender=0, age=25],
  Employee [id=15, lastName=1老师, email=laoshi@tomxwd.top, gender=0, age=20],
  Employee [id=13, lastName=Marry, email=marry@tomxwd.top, gender=0, age=15]
]
```
使用orderBy是默认asc（升序）。
<br>
使用orderAsc()升序。
使用orderDesc()降序
也可以使用**last()**方法，手动在sql最后拼接，有sql注入的风险！这里先使用orderBy("age")，再使用last("desc");
<br>
在last里面还可以做分页——last("desc limit 1,3")

---
## 使用Condition的方式打开如上需求
tbl_employee表中，年龄在18-50且为男性，姓名为Ken的所有用户<br>
相关代码：
```
/**
 * 用Condition来解决上述问题。
 * tbl_employee表中，年龄在18-50且为男性，姓名为Ken的所有用户
 * @throws Exception
 */
@SuppressWarnings("unchecked")
@Test
public void testCondition() throws Exception {
  List<Employee> emps = employeeMapper.selectPage(new Page<Employee>(1,2), Condition.create().between("age", 18, 50).eq("gender", 1).eq("last_name", "Ken"));
  System.out.println(emps);
}
```
日志输出：
```
DEBUG 06-09 13:07:37,324 ==>  Preparing: SELECT id,last_name AS lastName,email,gender,age FROM tbl_employee WHERE (age BETWEEN ? AND ? AND gender = ? AND last_name = ?)  (JakartaCommonsLoggingImpl.java:54)
DEBUG 06-09 13:07:37,354 ==> Parameters: 18(Integer), 50(Integer), 1(Integer), Ken(String) (JakartaCommonsLoggingImpl.java:54)
```
控制台输出：
```
[
  Employee [id=8, lastName=Ken, email=ken@tomxwd.top, gender=1, age=30],
  Employee [id=12, lastName=Ken, email=ken@tomxwd.top, gender=1, age=30]
]
```
---
## 总结：
MP：EntityWrapper&&Condition条件构造器。
<br>
MyBatis MGB(逆向工程):XxxExample-->Criteria:QBC风格（Query ByCriteria）
<br>
Hibernate、通用Mapper也是有QBC风格的。
MP中实现与QBC较为相似。
