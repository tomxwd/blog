---
title: 03_Mybatis-Plus入门CRUD
date: 2019-06-08 ‏‎‏‎17:05:13
tags: 
 - Java
 - Mybatis
 - Mybatis-Plus
categories:
 - Mybatis-Plus
---

# 03_Mybatis-Plus入门CRUD

---
## 通用的CRUD（入门DEMO）
1. 提出问题：
<br>
假设我们已经存在一张tbl_employee表，且已有对应的实体类Employee，实现tbl_employee表的CRUD操作我们需要做什么？
2. 实现方式：
<br>
  - 基于Mybatis：
    - 需要编写EmployeeMapper接口，并手动编写CRUD方法。
    - 提供EmployeeMapper.xml映射文件，并手动编写每个方法对应的SQL语句。
  - 基于MP
    - 只需要创建EmployeeMapper接口，并继承BaseMapper接口，这就是使用MP需要完成的所有操作，甚至不用创建SQL映射文件。
3. 具体实现
<br>首先要编写Mapper接口：
```java
package top.tomxwd.mp.mapper;
import com.baomidou.mybatisplus.mapper.BaseMapper;
import top.tomxwd.mp.beans.Employee;
/**
 * Mapper接口
 *
 * 基于Mybatis实现：在Mapper接口中编写CRUD相关方法，还要提供对应的SQL映射文件 以及方法对应的sql语句
 *
 * 基于MP：让XxxMapper接口继承BaseMapper接口即可
 * 			BaseMapper<T> : 泛型指定的就是当前Mapper接口所操作的实体类类型
 *
 */
public interface EmployeeMapper extends BaseMapper<Employee> {
}
```
**第二步要在实体类上的主键打@TabledId注解**<br>
```java
/**
 * @TableId：
 * 	value：指定表中的主键列的列名，如果实体属性名和列名一致，可以省略
 * 	type：指定主键策略
 */
@TableId(value="id",type=IdType.AUTO)
private Integer id;
```
**第三步要在实体类上打@TableName注解**

```java
/*
 * Mybatis-Plus会默认使用实体类的类名到数据库中找对应的表
 */
@TableName(value="tbl_employee")
public class Employee {}
```
以上就是MP的CRUD操作所需要做的事情。
4. 测试：
```java
private ApplicationContext ioc = new ClassPathXmlApplicationContext("applicationContext.xml");
private EmployeeMapper employeeMapper = ioc.getBean("employeeMapper",EmployeeMapper.class);
/**
 * 通用的插入操作
 * @throws Exception
 */
@Test
public void testCommonInsert() throws Exception {
  // 初始化Employee对象
  Employee employee = new Employee();
  employee.setAge(18);
  employee.setEmail("marry@tomxwd.top");
  employee.setGender(0);
  employee.setLastName("Marry");
  // 插入到数据库 返回受影响的条数
  Integer result = employeeMapper.insert(employee);
  System.out.println("result:" + result);
}
```
输出结果：
```
省略其他日志信息之后：
DEBUG 06-08 17:41:51,646 ==>  Preparing: INSERT INTO tbl_employee ( last_name, email, gender, age ) VALUES ( ?, ?, ?, ? )  (JakartaCommonsLoggingImpl.java:54)
DEBUG 06-08 17:41:51,679 ==> Parameters: Marry(String), marry@tomxwd.top(String), 0(Integer), 18(Integer) (JakartaCommonsLoggingImpl.java:54)
DEBUG 06-08 17:41:51,683 <==    Updates: 1 (JakartaCommonsLoggingImpl.java:54)
result:1
```
---

## 主键策略以及@TableId注解
|值|描述|
|:-:|:-:|
|idType.AUTO|数据库ID自增|
|idType.INPUT|用户输入ID|
|idType.ID_WORKER|全局唯一ID，内容为空自动填充**（默认配置）**|
|idType.UUID|全局唯一id，内容为空自动填充|
```java
/**
 * @TableId：
 * 	value：指定表中的主键列的列名，如果实体属性名和列名一致，可以省略
 * 	type：指定主键策略
 */
@TableId(value="id",type=IdType.AUTO)
private Integer id;
```
如果实体类field名和表中的字段名一样，则value可以不写。

---
## @TableName注解
表名注解。

|值|描述|
|:-:|:-:|
|value|表名（默认空）|
|resultMap|xml字段映射resultMap的ID|

---
## @TableField注解
字段注解：
<br>**其一：（value）**比如遇到表中是last_name，实体类属性是lastName，而且全局策略配置没有进行配置，则需要用到该字段，与@TableName类似，都是用来解决不统一的问题。用value来指定表中的字段名。
<br>**其二：（exist）**里面有个exist属性，表示这个是不是数据库表里面的字段，因为以后会有这种情况。

---
## MP全局策略配置(重要)
问题引出：
<br>
数据库中的字段名是last_name，而实体类属性名为lastName，但是根据日志信息得到的插入信息确实是last_name字段，为什么？
<br>
而且在mybatis-config.xml文件里也没有配置对应的驼峰规则等，为什么？
<br>
说明MP帮我们去解决了这个问题。
#### 定义Mybatis-Plus的全局策略配置
在spring配置文件（applicationContext.xml）中进行配置。
#### 为什么要进行全局配置？
比如每个实体类都以tbl_开头，那么每一个实体类都需要去声明@TableName，每一个主键id都需要配置自动递增，则需要每一个id字段去设置@TabledId注解。相对麻烦，如果用全局配置，则一步到位。
1. 字段驼峰原则
2. 配置全局主键策略
3. 配置全局表前缀策略
4. 后续会使用更多的策略来配置

```
<!-- 定义Mybatis-Plus的全局策略配置 -->
<!-- 定义Mybatis-Plus的全局策略配置 -->
<bean id="globalConfiguration" class="com.baomidou.mybatisplus.entity.GlobalConfiguration">
  <!-- 2.3版本之后，dbColumnUnderline默认就是true -->
  <!-- 驼峰原则 -->
  <property name="dbColumnUnderline" value="true"></property>
  <!-- 配置全局主键策略 0为自动递增 -->
  <property name="idType" value="0"></property>
  <!-- 全局的表前缀策略配置 -->
  <property name="tablePrefix" value="tbl_"></property>
</bean>
```
切记，在配置完全局策略配置之后，需要把改配置注入到Mybatis配置中去，也就是SqlSessionFactorybean，此时才会生效。否则报错。
```
<!-- 配置MybatisSqlSessionFactoryBean -->
<bean id="sqlSessionFactoryBean" class="com.baomidou.mybatisplus.spring.MybatisSqlSessionFactoryBean">
  <!-- 数据源 -->
  <property name="dataSource" ref="dataSource"></property>
  <property name="configLocation" value="classpath:mybatis-config.xml"></property>
  <!-- 别名处理 -->
  <property name="typeAliasesPackage" value="top.tomxwd.mp.beans"></property>
  <!-- 注入全局MP策略配置  -->
  <property name="globalConfig" ref="globalConfiguration"></property>
</bean>
```
---
## 插入数据获取主键值
**以前的做法：**<br>
```
Integer insertEmployee(Employee employee)
```
```
<insert useGeneratedKeys="true" keyProperty="id">SQL...</insert>
```
**MP的做法：**<br>
MP会自动把主键值回写到实体类中。

---
## insertAllColumn方法
普通的insert方法是使用非空来判断是否插入值，而insertAllColumn方法是插入所有的字段。
```
/**
 * insertAllColumn方法
 * @throws Exception
 */
@Test
public void testInsertAllColumn() throws Exception {
  Employee e = new Employee();
  e.setLastName("Tony");
  e.setEmail("tony@tomxwd.top");
  e.setAge(50);
  Integer result = employeeMapper.insertAllColumn(e);
  System.out.println(result);
  Integer id = e.getId();
  System.out.println(id);
}
```
此时日志记录为：
```
DEBUG 06-08 20:33:09,786 ==>  Preparing: INSERT INTO tbl_employee ( last_name,email,gender,age ) VALUES ( ?,?,?,? )  (JakartaCommonsLoggingImpl.java:54)
DEBUG 06-08 20:33:09,820 ==> Parameters: Tony(String), tony@tomxwd.top(String), null, 50(Integer) (JakartaCommonsLoggingImpl.java:54)
DEBUG 06-08 20:33:09,824 <==    Updates: 1 (JakartaCommonsLoggingImpl.java:54)
```
发现数据库结果：gender为null，没有用到数据库中的default值。

---
## updateById方法
**null则不更新**，返回结果是影响条数。
<br>
相关代码：
```
/**
 * 通用更新操作
 * @throws Exception
 */
@Test
public void testUpdateById() throws Exception {
  Employee e = new Employee();
  e.setId(9);
  e.setGender(1);
  Integer result = employeeMapper.updateById(e);
  System.out.println(result);
}
```
日志输出：
```
DEBUG 06-08 20:38:33,846 ==>  Preparing: UPDATE tbl_employee SET gender=? WHERE id=?  (JakartaCommonsLoggingImpl.java:54)
DEBUG 06-08 20:38:33,888 ==> Parameters: 1(Integer), 9(Integer) (JakartaCommonsLoggingImpl.java:54)
DEBUG 06-08 20:38:33,890 <==    Updates: 1 (JakartaCommonsLoggingImpl.java:54)
```

---
## updateAllColumnById方法
跟updateById的区别在于，**null值也会插入到里面。**<br>
相关代码：
```
/**
 * UpdateAllColumnById方法
 * @throws Exception
 */
@Test
public void testUpdateAllColumnById() throws Exception {
  Employee e = new Employee();
  e.setId(9);
  e.setGender(1);
  Integer result = employeeMapper.updateAllColumnById(e);
  System.out.println(result);
}
```
日志输出：
```
DEBUG 06-08 20:44:18,682 ==>  Preparing: UPDATE tbl_employee SET last_name=?,email=?,gender=?,age=? WHERE id=?  (JakartaCommonsLoggingImpl.java:54)
DEBUG 06-08 20:44:18,742 ==> Parameters: null, null, 1(Integer), null, 9(Integer) (JakartaCommonsLoggingImpl.java:54)
DEBUG 06-08 20:44:18,746 <==    Updates: 1 (JakartaCommonsLoggingImpl.java:54)
```

---
## selectById方法
相关代码：
```
/**
 * selectById方法
 * @throws Exception
 */
@Test
public void testSelectById() throws Exception {
  Employee employee = employeeMapper.selectById(8);
  System.out.println(employee);
}
```
日志输出:
```
DEBUG 06-08 20:48:39,273 ==>  Preparing: SELECT id,last_name AS lastName,email,gender,age FROM tbl_employee WHERE id=?  (JakartaCommonsLoggingImpl.java:54)
DEBUG 06-08 20:48:39,308 ==> Parameters: 8(Integer) (JakartaCommonsLoggingImpl.java:54)
DEBUG 06-08 20:48:39,326 <==      Total: 1 (JakartaCommonsLoggingImpl.java:54)
```

---
## selectOne方法（条件查询）
相关代码：
```
/**
 * 多个列来查询
 * SelectOne
 * @throws Exception
 */
@Test
public void testSelectOne() throws Exception {
  // 通过多个列进行查询 比如 id和lastName来查询
  Employee employee = new Employee();
  employee.setId(7);
  employee.setLastName("Marry");
  Employee e = employeeMapper.selectOne(employee);
  System.out.println(e);
}
```
日志输出:
```
DEBUG 06-08 20:52:39,303 ==>  Preparing: SELECT id,last_name AS lastName,email,gender,age FROM tbl_employee WHERE id=? AND last_name=?  (JakartaCommonsLoggingImpl.java:54)
DEBUG 06-08 20:52:39,348 ==> Parameters: 7(Integer), Marry(String) (JakartaCommonsLoggingImpl.java:54)
DEBUG 06-08 20:52:39,364 <==      Total: 1 (JakartaCommonsLoggingImpl.java:54)
```
需要注意的是！该方法是查询一条数据的，不可以返回多个数据，否则报错！！！

---
## selectBatchIds方法（批量查询根据ID）
相关代码：
```
/**
 * 批量查询，根据Id
 * @throws Exception
 */
@Test
public void testSelectBatchIds() throws Exception {
  ArrayList<Integer> Ids = new ArrayList<Integer>();
  Ids.add(1);
  Ids.add(2);
  Ids.add(3);
  List<Employee> list = employeeMapper.selectBatchIds(Ids);
  System.out.println(list);
}
```
日志输出：
```
DEBUG 06-08 21:06:49,409 ==>  Preparing: SELECT id,last_name AS lastName,email,gender,age FROM tbl_employee WHERE id IN ( ? , ? , ? )  (JakartaCommonsLoggingImpl.java:54)
DEBUG 06-08 21:06:49,440 ==> Parameters: 1(Integer), 2(Integer), 3(Integer) (JakartaCommonsLoggingImpl.java:54)
DEBUG 06-08 21:06:49,457 <==      Total: 3 (JakartaCommonsLoggingImpl.java:54)
```
控制台输出：
```
[
  Employee [id=1, lastName=Tom, email=tom@tomxwd.top, gender=1, age=22],
  Employee [id=2, lastName=Jerry, email=jerry@tomxwd.top, gender=0, age=25],
  Employee [id=3, lastName=Black, email=black@tomxwd.top, gender=1, age=30]
]
```

## selectByMap方法（根据Map对象进行条件查询）
需要注意的是，Map中封装的key是column名，也就是数据库中的列名，而不是实体类的属性名。
相关代码：
```
/**
 * 根据Map对象查询，即是封装为Map对象，条件查询
 * @throws Exception
 */
@Test
public void testSelectByMap() throws Exception {
  Map<String,Object> columnMap = new HashMap<String, Object>();
  columnMap.put("last_name", "Marry");
  List<Employee> list = employeeMapper.selectByMap(columnMap);
  System.out.println(list);
}
```
日志输出：
```
DEBUG 06-08 21:13:16,779 ==>  Preparing: SELECT id,last_name AS lastName,email,gender,age FROM tbl_employee WHERE last_name = ?  (JakartaCommonsLoggingImpl.java:54)
DEBUG 06-08 21:13:16,808 ==> Parameters: Marry(String) (JakartaCommonsLoggingImpl.java:54)
DEBUG 06-08 21:13:16,824 <==      Total: 3 (JakartaCommonsLoggingImpl.java:54)
```
控制台输出：
```
[Employee [id=5, lastName=Marry, email=marry@tomxwd.top, gender=0, age=18], Employee [id=6, lastName=Marry, email=marry@tomxwd.top, gender=0, age=18], Employee [id=7, lastName=Marry, email=marry@tomxwd.top, gender=0, age=18]]
```

---
## selectPage方法（分页查询）
第一个参数：Page对象，继承了Pagination对象，而Pagination对象又继承了RowBounds对象（Mybatis的分页对象）
<br>第二个参数是条件对象，后续说明，此处不用。
相关代码：
```
/**
 * 分页查询
 * 第一个对象为Page对象，继承了Pagination对象，而Pagination对象继承RowBounds（Mybatis的分页对象）
 * 第二个对象是条件对象
 * @throws Exception
 */
@Test
public void testSelectPage() throws Exception {
  List<Employee> list = employeeMapper.selectPage(new Page<Employee>(2, 2), null);
  System.out.println(list);
}
```
日志输出：
```
DEBUG 06-08 21:20:29,110 ==>  Preparing: SELECT id,last_name AS lastName,email,gender,age FROM tbl_employee  (JakartaCommonsLoggingImpl.java:54)
DEBUG 06-08 21:20:29,181 ==> Parameters:  (JakartaCommonsLoggingImpl.java:54)
```
控制台输出：
```
[
  Employee [id=3, lastName=Black, email=black@tomxwd.top, gender=1, age=30],
  Employee [id=4, lastName=White, email=white@tomxwd.top, gender=0, age=35]
]
```
根据日志信息可知，并没有看到limit子句，说明底层还是用了Mybatis提供的RowBounds分页。**使用的是内存的分页方式。**<br>
如果想要做到真正的物理分页（limit）还是要依赖一些分页插件来做（**PageHelper等**）。

---
## deleteById方法（根据id删除）
相关代码：
```
/**
 * 根据id删除数据
 * @throws Exception
 */
@Test
public void testDeleteById() throws Exception {
  Integer result = employeeMapper.deleteById(9);
  System.out.println(result);
}
```
日志输出：
```
DEBUG 06-08 21:30:41,429 ==>  Preparing: DELETE FROM tbl_employee WHERE id=?  (JakartaCommonsLoggingImpl.java:54)
DEBUG 06-08 21:30:41,470 ==> Parameters: 9(Integer) (JakartaCommonsLoggingImpl.java:54)
DEBUG 06-08 21:30:41,473 <==    Updates: 1 (JakartaCommonsLoggingImpl.java:54)
```
控制台输出：
```
1
```

---
## deleteByMap方法（map来封装列名信息）
map的key是列名，而不是实体类的属性名。<br>
相关代码：
```
/**
 * 根据columnMap条件删除数据
 * @throws Exception
 */
@Test
public void testDeleteByMap() throws Exception {
  Map<String, Object> columnMap = new HashMap<String, Object>();
  columnMap.put("last_name", "Marry");
  columnMap.put("gender", 0);
  Integer result = employeeMapper.deleteByMap(columnMap);
  System.out.println(result);
}
```
日志输出：
```
DEBUG 06-08 21:35:50,642 ==>  Preparing: DELETE FROM tbl_employee WHERE gender = ? AND last_name = ?  (JakartaCommonsLoggingImpl.java:54)
DEBUG 06-08 21:35:50,674 ==> Parameters: 0(Integer), Marry(String) (JakartaCommonsLoggingImpl.java:54)
DEBUG 06-08 21:35:50,677 <==    Updates: 3 (JakartaCommonsLoggingImpl.java:54)
```
控制台输出：
```
3
```

## deleteBatchIds方法（根据Id批量删除）
相关代码：
```
/**
 * 根据Id批量删除
 * @throws Exception
 */
@Test
public void testDeleteBatchIds() throws Exception {
  List<Integer> Ids = new ArrayList<Integer>();
  Ids.add(10);
  Ids.add(11);
  Integer result = employeeMapper.deleteBatchIds(Ids);
  System.out.println(result);
}
```
日志输出：
```
DEBUG 06-08 21:43:24,751 ==>  Preparing: DELETE FROM tbl_employee WHERE id IN ( ? , ? )  (JakartaCommonsLoggingImpl.java:54)
DEBUG 06-08 21:43:24,838 ==> Parameters: 10(Integer), 11(Integer) (JakartaCommonsLoggingImpl.java:54)
DEBUG 06-08 21:43:24,840 <==    Updates: 2 (JakartaCommonsLoggingImpl.java:54)
```
控制台输出：
```
2
```

---
## MP启动注入SQL原理分析
#### 问题：
XxxMapper继承了BaseMapper<T>,BaseMapper中提供了通用的CRUD方法，方法来源于BaseMapper，有方法就必须有SQL，因为Mybatis最终还是要通过SQL语句来操作数据。
<br>
###### 前置知识：
Mybatis源码中有比较重要的一些对象，Mybatis框架的执行流程。
- Configuration 全局配置对象
- MappedStatement
- ......

#### 通过现象看到本质
1. employeeMapper的本质：org.apache.ibatis.binding.MapperProxy
2. MapperProxy中，有一个sqlSession-->SqlSessionFactory。
3. SqlSessionFactory-->Configuration-->MappedStatements，**（每一个MappedStatement都表示Mapper接口中的一个方法与Mapper映射文件中的一个SQL）**。在这里面-->sqlSource-->sql里面就是要执行的sql语句。
<br>
也就是说，MP在启动的时候会挨个分析XxxMapper中的方法，并且将对应的SQL语句处理好，保存到Configuration对象的MappedStatements中。

##### 本质
对象介绍：
1. sqlMethod:枚举对象，MP支持的SQL方法
2. TableInfo:数据库表反射信息，可以获取到数据库表相关的信息。
3. sqlSource:SQL语句处理对象
4. MapperBuilderAssistant:用于缓存、SQL参数、查询返回结果集处理等。<br>
通过MapperBuilderAssistant添加到Configrration中的mappedstatements中去。


```
DEBUG 06-08 21:43:24,522 Invoking afterPropertiesSet() on bean with name 'employeeMapper' (AbstractAutowireCapableBeanFactory.java:1670)
DEBUG 06-08 21:43:24,567 addMappedStatement: top.tomxwd.mp.mapper.EmployeeMapper.deleteById (JakartaCommonsLoggingImpl.java:54)
DEBUG 06-08 21:43:24,570 addMappedStatement: top.tomxwd.mp.mapper.EmployeeMapper.deleteBatchIds (JakartaCommonsLoggingImpl.java:54)
DEBUG 06-08 21:43:24,573 addMappedStatement: top.tomxwd.mp.mapper.EmployeeMapper.updateById (JakartaCommonsLoggingImpl.java:54)
DEBUG 06-08 21:43:24,575 addMappedStatement: top.tomxwd.mp.mapper.EmployeeMapper.updateAllColumnById (JakartaCommonsLoggingImpl.java:54)
DEBUG 06-08 21:43:24,575 addMappedStatement: top.tomxwd.mp.mapper.EmployeeMapper.selectById (JakartaCommonsLoggingImpl.java:54)
DEBUG 06-08 21:43:24,576 addMappedStatement: top.tomxwd.mp.mapper.EmployeeMapper.selectBatchIds (JakartaCommonsLoggingImpl.java:54)
DEBUG 06-08 21:43:24,579 addMappedStatement: top.tomxwd.mp.mapper.EmployeeMapper.insert (JakartaCommonsLoggingImpl.java:54)
DEBUG 06-08 21:43:24,581 addMappedStatement: top.tomxwd.mp.mapper.EmployeeMapper.insertAllColumn (JakartaCommonsLoggingImpl.java:54)
DEBUG 06-08 21:43:24,584 addMappedStatement: top.tomxwd.mp.mapper.EmployeeMapper.delete (JakartaCommonsLoggingImpl.java:54)
DEBUG 06-08 21:43:24,586 addMappedStatement: top.tomxwd.mp.mapper.EmployeeMapper.deleteByMap (JakartaCommonsLoggingImpl.java:54)
DEBUG 06-08 21:43:24,588 addMappedStatement: top.tomxwd.mp.mapper.EmployeeMapper.update (JakartaCommonsLoggingImpl.java:54)
DEBUG 06-08 21:43:24,590 addMappedStatement: top.tomxwd.mp.mapper.EmployeeMapper.updateForSet (JakartaCommonsLoggingImpl.java:54)
DEBUG 06-08 21:43:24,592 addMappedStatement: top.tomxwd.mp.mapper.EmployeeMapper.selectByMap (JakartaCommonsLoggingImpl.java:54)
DEBUG 06-08 21:43:24,602 addMappedStatement: top.tomxwd.mp.mapper.EmployeeMapper.selectOne (JakartaCommonsLoggingImpl.java:54)
DEBUG 06-08 21:43:24,604 addMappedStatement: top.tomxwd.mp.mapper.EmployeeMapper.selectCount (JakartaCommonsLoggingImpl.java:54)
DEBUG 06-08 21:43:24,608 addMappedStatement: top.tomxwd.mp.mapper.EmployeeMapper.selectList (JakartaCommonsLoggingImpl.java:54)
DEBUG 06-08 21:43:24,610 addMappedStatement: top.tomxwd.mp.mapper.EmployeeMapper.selectPage (JakartaCommonsLoggingImpl.java:54)
DEBUG 06-08 21:43:24,612 addMappedStatement: top.tomxwd.mp.mapper.EmployeeMapper.selectMaps (JakartaCommonsLoggingImpl.java:54)
DEBUG 06-08 21:43:24,614 addMappedStatement: top.tomxwd.mp.mapper.EmployeeMapper.selectMapsPage (JakartaCommonsLoggingImpl.java:54)
DEBUG 06-08 21:43:24,616 addMappedStatement: top.tomxwd.mp.mapper.EmployeeMapper.selectObjs (JakartaCommonsLoggingImpl.java:54)
```
从日志信息中可以看出，在启动的时候，会把每一个方法构建成每个MappedStatement。
**整个构造流程**
1. AutoSqlInjector（SQL自动注入器）
2. injectDeleteByIdSql方法构建出SqlSource（languageDriver.createSqlSource(configuration,sql,modelClass;)
3. this.addDeleteMappedStatement(mapperClass,sqlMethod.getMethod(),sqlSource)
4. builderAssistant.addMappedStatement->configuration.addMappedStatement(statement);最终注入到configuration中。

---
## 总结
1. 以上是基本的CRUD操作，我们仅仅需要继承一个BaseMapper接口，就可以实现大部分的单标CRUD操作。BaseMapper提供了多达17个方法给我们使用，可以很方便的实现单一、批量、分页等操作。极大减少开发负担，难道这就是MP的强大之处吗？
2. 提出需求：
<br>现在有一个需求，我们需要分页查询Employee表中，年龄在18-50之间性别为难而且姓名为Xx的用户，这时候我们怎么实现上述需求呢？
<br>
Mybatis:需要在SQL映射文件中编写带有条件查询的SQL，并且基于PageHelper插件完成分页。
<br>实现上面一个简单的需求，往往需要我们做很多重复单调的工作。普通的Mapper能够解决这类痛点吗？
<br>
MP：依旧不用编写SQL语句，MP提供了功能强大的条件构造器EntityWrapper。
