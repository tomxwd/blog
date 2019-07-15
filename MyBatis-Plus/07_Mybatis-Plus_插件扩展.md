---
title: 07_Mybatis-Plus_插件扩展
data: 2019-06-15 ‏‎‏‎15:09:29
tags: 
 - Java
 - Mybatis
 - Mybatis-Plus
categories:
 - Mybatis
 - Mybatis-Plus
---

# 07_Mybatis-Plus_插件扩展

## Mybatis插件机制简介

#### 插件机制：

Mybatis通过插件（Interceptor）可以做到拦截四大对象相关方法的执行，根据需求，完成数据的动态改变。

- Executor

- StatementHandler
- ParamerterHandler
- ResultSetHandler

#### 插件原理：

四大对象的每个对象在创建的时候，都会执行interceptorChain.pluginAll()，会经过每个插件对象的plugin()方法，目的是为当前的四大对象创建代理。代理对象就可以拦截到四大对象相关的方法的执行，因为要执行四大对象的方法需要经过代理。

---

## 分页插件

#### com.baomidou.mybatisplus.plugins.PaginationInterceptor

在spring配置文件的mybatis配置中添加：

```xml
<!-- 注册插件 -->
<property name="plugins">
    <list>
        <!-- 注册分页插件 -->
        <bean class="com.baomidou.mybatisplus.plugins.PaginationInterceptor"></bean>
    </list>
</property>
```

###### 测试：

相关代码：

```java
public class TestMP {
	
	ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
	
	EmployeeMapper employeeMapper = ctx.getBean("employeeMapper",EmployeeMapper.class);
	
	/**
	 * 测试分页插件
	 * @throws Exception
	 */
	@Test
	public void testPage() throws Exception {
		List<Employee> emps = employeeMapper.selectPage(new Page<Employee>(1, 1), null);
		System.out.println(emps);
	}
	
}
```

输出日志：

```
DEBUG 06-15 15:25:34,428 ==>  Preparing: SELECT COUNT(1) FROM tbl_employee  (BaseJdbcLogger.java:159)
DEBUG 06-15 15:25:34,452 ==> Parameters:  (BaseJdbcLogger.java:159)
DEBUG 06-15 15:25:34,484 ==>  Preparing: SELECT id AS id,last_name AS lastName,email,gender,age FROM tbl_employee LIMIT 0,1  (BaseJdbcLogger.java:159)
DEBUG 06-15 15:25:34,485 ==> Parameters:  (BaseJdbcLogger.java:159)
DEBUG 06-15 15:25:34,489 <==      Total: 1 (BaseJdbcLogger.java:159)
```

控制台输出：

```
[Employee{, id=1, lastName=Tom, email=tom@tomxwd.top, gender=1, age=22}]
```

可以看到是有物理分页效果的，LIMIT子句。

## 分页后Page对象的使用

相关代码：

```java
public class TestMP {
	
	ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
	
	EmployeeMapper employeeMapper = ctx.getBean("employeeMapper",EmployeeMapper.class);
	
	/**
	 * 测试分页插件
	 * @throws Exception
	 */
	@Test
	public void testPage() throws Exception {
		Page<Employee> page = new Page<Employee>(1,1);
		List<Employee> emps = employeeMapper.selectPage(page, null);
		System.out.println("---------------");
		System.out.println("总条数："+page.getTotal());
		System.out.println("当前页码："+page.getCurrent());
		System.out.println("总页码："+page.getPages());
		System.out.println("每页显示的条数："+page.getSize());
		System.out.println("是否有上一页："+page.hasPrevious());
		System.out.println("是否有下一页："+page.hasNext());
        
        // 将查询的结果封装到page对象中
		page.setRecords(emps);
		System.out.println(page.getRecords());
	}
	
}
```

控制台输出：

```
总条数：6
当前页码：1
总页码：6
每页显示的条数：1
是否有上一页：false
是否有下一页：true
[Employee{, id=1, lastName=Tom, email=tom@tomxwd.top, gender=1, age=22}]
```

## 执行分析插件

1. com.baomidou.mybatisplus.plugins.SqlExplainInterceptor
2. SQL执行分析拦截器，只支持**MySQL5.6.3以上**版本
3. 该插件的作用是分析DELETE、UPDATE语句，防止小白或者恶意进行DELTE、UPDATE全表操作
4. 只建议在开发环境中使用，不建议在生产环境使用
5. 在插件的底层通过SQL语句分析命令：Explain分析当前的SQL语句，根据结果集中的Extra列来断定当前是否全表操作

首先在Spring配置文件中需要加入执行分析插件：

```
<!-- 注册插件 -->
<property name="plugins">
    <list>
    <!-- 注册执行分析插件 -->
        <bean class="com.baomidou.mybatisplus.plugins.SqlExplainInterceptor">
        	<property name="stopProceed" value="true"></property>
        </bean>
    </list>
</property>
```

日志输出：

```
WARN  06-15 15:50:16,300 Warn: Your mysql version needs to be greater than '5.6.3' to execute of Sql Explain! (SqlExplainInterceptor.java:80)
DEBUG 06-15 15:50:16,319 ==>  Preparing: DELETE FROM tbl_employee  (BaseJdbcLogger.java:159)
DEBUG 06-15 15:50:16,345 ==> Parameters:  (BaseJdbcLogger.java:159)
DEBUG 06-15 15:50:16,349 <==    Updates: 6 (BaseJdbcLogger.java:159)
```

由于本机使用的版本不是5.6.3以上的，所以没能成功。

若正常情况，则会报错，表示你正在执行删除全表操作。

原理是使用MySQL中的`EXPLAIN`关键字，如EXPLAIN DELETE FROM tbl_employee，执行后有个Extra字段，结果可能是Deleting all rows或者Using where等，如果分析出不是Using where，则抛出异常。

## 性能分析插件

1. com.baomidou.mybatisplus.plugins.PerformanceInterceptor
2. 性能分析拦截器，用于输出每条SQL语句以及其执行时间
3. SQL性能执行分析，**开发环境使用**，超过指定时间，停止运行，有利于发现问题

首先在Spring配置文件中注册插件：

```xml
    	<!-- 注册插件 -->
    	<property name="plugins">
    		<list>
    			<!-- 注册性能分析插件 -->
    			<bean class="com.baomidou.mybatisplus.plugins.PerformanceInterceptor">
    				<!--   -->
    				<property name="maxTime" value="100"></property>
    				<!-- SQL是否格式化 默认false -->
    				<property name="format" value="true"></property>
    			</bean>
    		</list>
    	</property>
```

日志输出会多出这段红色的文本：

正常输出：

```
 Time：16 ms - ID：top.tomxwd.mp.mapper.EmployeeMapper.insert
 Execute SQL：
    INSERT 
    INTO
        tbl_employee
        ( last_name, email, gender, age ) 
    VALUES
        ( '12', '12123@qq.com', '1', 22 )]
```

超时输出：

```
未起作用。。。
否则会报错，超时异常。
```

## 乐观锁插件

1. com.baomidou.mybatisplus.plugins.OptimisticLockerInterceptor
2. 如果想实现如下需求：
   - 当要更新一条记录的时候，希望这条记录没有被别人更新
3. 乐观锁的实现原理：
   - 取出记录的时候，获取当前version
   - 更新时，带上这个version
   - 执行更新时，set version = yourVersion+1 where version = yourVersion
   - 如果version不对，就更新失败
4. @Version用于注解实体字段，必须要有

首先注册插件：

```xml
<!-- 注册乐观锁插件 -->
<bean class="com.baomidou.mybatisplus.plugins.OptimisticLockerInterceptor"></bean>
```

加一个字段表示version。

加一个属性表示version，并打上@Version注解。

相关代码：

```java
/**
* 测试 乐观锁插件
* @throws Exception
*/
@Test
public void testOptimisticLocker() throws Exception {
    // 更新操作
    Employee emp = new Employee();
    emp.setId(13);
    emp.setLastName("test");
    emp.setVersion(1);
    employeeMapper.updateById(emp);
}
```



日志输出：

```
DEBUG 06-15 19:32:47,513 ==>  Preparing: UPDATE tbl_employee SET last_name=?, version=? WHERE id=? and version=?  (BaseJdbcLogger.java:159)
DEBUG 06-15 19:32:47,550 ==> Parameters: test(String), 2(Integer), 13(Integer), 1(Integer) (BaseJdbcLogger.java:159)
DEBUG 06-15 19:32:47,553 <==    Updates: 1 (BaseJdbcLogger.java:159)

```

重点关注sql语句：`UPDATE tbl_employee SET last_name=?, version=? WHERE id=? and version=?`

以及参数的值`test(String), 2(Integer), 13(Integer), 1(Integer) `

1. 发现version不但被更新，并且在where条件中。
2. 发现set的version值为2，而where的version值为1，说明在对象设置属性值的时候，version指的是数据库里的版本。



