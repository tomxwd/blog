---
title: 08_自定义全局操作以及全局Sql注入器
date: 2019-06-15 ‏‎‏‎‏‎19:39:11
tags: 
 - Java
 - Mybatis
 - Mybatis-Plus
categories:
 - Mybatis-Plus
---

# 08_自定义全局操作以及全局Sql注入器

根据MybatisPlus的AutoSQLInjector可以自定义各种你想要的sql，注入到全局中，相当于自定义MybatisPlus自动注入的方法。

之前需要在xml中进行配置的SQL语句，现在通过扩展AutoSqlInjector在加载mybatis环境时就注入。

## AutoSqlInjector

1. 在Mapper接口定义相关的CRUD方法。
2. 扩展AutoSQLInjector的inject方法，实现Mapper接口中方法要注入的SQL。
3. 在MP全局策略中，配置自定义注入器。

首先在mapper接口中加入deleteAll()方法：

```java
/**
 * <p>
 *  Mapper 接口
 * </p>
 *
 * @author weidu.xie
 * @since 2019-06-09
 */
public interface EmployeeMapper extends BaseMapper<Employee> {
	
	int deleteAll();
	
}
```

第二步定义一个继承AutoSqlInjector类的类，并扩展inject方法，完成自定义全局操作

```java
/**
 * 自定义全局操作
 */
public class MySqlInjector extends AutoSqlInjector{
	
	/**
	 * 扩展inject方法 ，完成自定义全局操作
	 */
	@Override
	public void inject(Configuration configuration, MapperBuilderAssistant builderAssistant, Class<?> mapperClass,
			Class<?> modelClass, TableInfo table) {
		// 将EmployeeMapper中定义的deleteAll方法，处理成对应的MappedStatement对象，加入到Configuration对象中。
	}
	
}
```

最后在配置文件中声明该类，并注入到全局配置中去。

```xml
   	<!-- 定义Mybatis-Plus的全局策略配置 -->
   	<bean id="globalConfiguration" class="com.baomidou.mybatisplus.entity.GlobalConfiguration">
   		<!-- 2.3版本之后，dbColumnUnderline默认就是true -->
   		<!-- 驼峰原则 -->
   		<property name="dbColumnUnderline" value="true"></property>
   		<!-- 配置全局主键策略 0为自动递增 -->
    	<property name="idType" value="0"></property>
    	<!-- 全局的表前缀策略配置 -->
    	<property name="tablePrefix" value="tbl_"></property>
    	<!-- 注入自定义全局操作 -->
    	<property name="sqlInjector" ref="mySqlInjector"></property>
   	</bean>
   	<!-- 定义自定义注入器 -->
   	<bean id="mySqlInjector" class="top.tomxwd.mp.injector.MySqlInjector"></bean>
    
```

#### 实际使用：

 inject方法的编写：

```java
/**
 * 自定义全局操作
 */
public class MySqlInjector extends AutoSqlInjector{
	
	/**
	 * 扩展inject方法 ，完成自定义全局操作
	 */
	@Override
	public void inject(Configuration configuration, MapperBuilderAssistant builderAssistant, Class<?> mapperClass,
			Class<?> modelClass, TableInfo table) {
		// 将EmployeeMapper中定义的deleteAll方法，处理成对应的MappedStatement对象，加入到Configuration对象中。
		
		// 注入的sql语句
		String sql = "delete from "+table.getTableName();
		// 注入的方法名,!!!!一定要与EmployeeMapper接口中的方法名一致
		String method = "deleteAll";
		// 构造SqlSource对象
		SqlSource sqlSource = languageDriver.createSqlSource(configuration, sql, modelClass);
		// 构造一个删除的MappedStatement
		this.addDeleteMappedStatement(mapperClass, method, sqlSource);
	}
	
}
```

测试代码：

```java
public class TestMP {

	ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
	
	EmployeeMapper employeeMapper = ctx.getBean("employeeMapper",EmployeeMapper.class);
	
	
	/**
	 * 测试自定义全局操作
	 * @throws Exception
	 */
	@Test
	public void testMySqlInjector() throws Exception {
		int result = employeeMapper.deleteAll();
		System.out.println("result:"+result);
	}
	
}
```

日志分析：

```
DEBUG 06-15 20:34:09,264 addMappedStatement: top.tomxwd.mp.mapper.WorkerMapper.deleteAll (MybatisConfiguration.java:67)
```

表示在启动的时候就注入了这个mappedStatement。

```
DEBUG 06-15 20:34:09,327 ==>  Preparing: delete from tbl_employee  (BaseJdbcLogger.java:159)
DEBUG 06-15 20:34:09,359 ==> Parameters:  (BaseJdbcLogger.java:159)
DEBUG 06-15 20:34:09,362 <==    Updates: 13 (BaseJdbcLogger.java:159)
```

控制台：

```
result:13
```



------

## 自定义注入器的应用之——逻辑删除

假删除、逻辑删除：并不会真正从数据库中将数据删除掉，而是将当前被删除的这条数据中的一个逻辑删除字段置为删除状态。

1. com.baomidou.mybatisplus.mapper.LogicSqlInjector
2. logicDeleteValue 逻辑删除全局值
3. logicNotDeleteValue 逻辑未删除全局值
4. 在POJO的逻辑删除字段，添加上@TableLogic注解
5. 会在mp自带查询和更新方法的sql后面，追加**逻辑删除字段**=LogicNotDeleteValue默认值
   - 删除方法：deleteById()和其他的delete方法，底层SQL调用的是update_tbl_xxx set **逻辑删除字段**=logicDeleteValue默认值。

首先在配置文件中注入LogicSQLInjector对象以及将其配置到全局配置中去。

```xml
  <!-- 定义Mybatis-Plus的全局策略配置 -->
  <bean id="globalConfiguration" class="com.baomidou.mybatisplus.entity.GlobalConfiguration">
   		<!-- 2.3版本之后，dbColumnUnderline默认就是true -->
   		<!-- 驼峰原则 -->
   		<property name="dbColumnUnderline" value="true"></property>
   		<!-- 配置全局主键策略 0为自动递增 -->
    	<property name="idType" value="0"></property>
    	<!-- 全局的表前缀策略配置 -->
    	<property name="tablePrefix" value="tbl_"></property>
    	<!-- 注入自定义全局操作 -->
    	<!-- <property name="sqlInjector" ref="mySqlInjector"></property> -->
    	<!-- 注入逻辑删除 -->
    	<property name="sqlInjector" ref="logicSqlInjector"></property>
   	</bean>
 <!-- 逻辑删除 -->
    <bean id="logicSqlInjector" class="com.baomidou.mybatisplus.mapper.LogicSqlInjector"></bean>
    
```

再在pojo对象上的逻辑删除字段上打注解@TableLogic

```java
@TableLogic //逻辑删除属性
private Integer valid;
```

在全局配置中说明删除状态的表示值。

```xml
<!-- 注入逻辑删除全局值 -->
<property name="logicDeleteValue" value="0"></property>
<property name="logicNotDeleteValue" value="1"></property>
```

测试分为两部分，一部分是查找数据，发现逻辑删除字段值为0的没有被找出来

日志输出：

```
DEBUG 06-15 21:17:13,702 ==>  Preparing: SELECT id,dept_name AS deptName,`valid` FROM tbl_dept WHERE `valid`=1  (BaseJdbcLogger.java:159)
DEBUG 06-15 21:17:13,733 ==> Parameters:  (BaseJdbcLogger.java:159)
DEBUG 06-15 21:17:13,758 <==      Total: 1 (BaseJdbcLogger.java:159)
```

发现在后面会拼上一段关于valid字段的判断。如果为1才会被筛选出来。

控制台输出：

```
[Dept2 [id=1, deptName=一部, valid=1]]
```

第二部分测试是关于删除：

日志输出：

```
DEBUG 06-15 21:18:32,992 ==>  Preparing: UPDATE tbl_dept SET `valid`=0 WHERE id=?  (BaseJdbcLogger.java:159)
DEBUG 06-15 21:18:33,021 ==> Parameters: 1(Integer) (BaseJdbcLogger.java:159)
DEBUG 06-15 21:18:33,023 <==    Updates: 1 (BaseJdbcLogger.java:159)
```

发现走的是Update语法，只是更改了valid的值。

控制台输出：

```
result:1
```

