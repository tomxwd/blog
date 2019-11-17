---
title: 06_Mybatis-Plus_代码生成器
date: 2019-06-09 15:34:47
tags: 
 - Java
 - Mybatis
 - Mybatis-Plus
categories:
 - Mybatis-Plus
---

# 06_Mybatis-Plus_代码生成器

---
## MP与MGB对比
1. MP提供了大量的自定义设置，生成的代码完全能够满足各类型的需求。
2. MP的代码生成器 和Mybatis MGB（逆向工程）代码生成器
  - MP的代码生成器都是基于java代码来生成。
  - MBG基于xml文件进行代码生成。
  - Mybatis的代码生成器可以生成：实体类、Mapper接口、Mapper映射文件。
  - MP的代码生成器可生成：实体类（**可以选择是否支持AR**）、Mapper接口、Mapper映射文件、**Service层、Controller层。**
---
## 表及字段命名策略选择
<br>
在MP中，我们建议数据库**表名**和**字段名**采用**驼峰命名**方式，如果采用下划线命名方式，则要开启下划线开关，如果表名字段名命名方式不一致，请注解指定，建议最好保持一致。
<br>
这么做的原因是为了**避免在对应实体类时产生的性能损耗**，这样字段不用做映射就能直接和实体类对应。当然如果项目里不用考虑这点性能损耗，那么可以采用下划线，只需要在生成代码时配置dbColumnUnderline属性即可。

---
## 代码生成器依赖
#### 模板引擎
MP的代码生成器默认使用的是Apache的Velocity模板，当然也可以更换为别的模板技术，例如freemarker。此处不做过多的介绍。
<br>加入**Apache Velocity依赖**
```
<!-- Apache Velocity模板引擎 -->
<dependency>
  <groupId>org.apache.velocity</groupId>
  <artifactId>velocity-engine-core</artifactId>
  <version>2.0</version>
</dependency>
```
加入slf4j，查看日志输出信息
```
<!-- slf4j 日志输出 -->
<dependency>
	<groupId>org.slf4j</groupId>
	<artifactId>slf4j-api</artifactId>
	<version>1.7.7</version>
</dependency>
<dependency>
	<groupId>org.slf4j</groupId>
	<artifactId>slf4j-log4j12</artifactId>
	<version>1.7.7</version>
</dependency>
```
---
#### 全局配置
相关代码：
```
//1.全局配置
GlobalConfig config = new GlobalConfig();
config.setActiveRecord(true)//是否支持AR模式
.setAuthor("weidu.xie")//设置作者
.setOutputDir("E:\\Java文件\\mp04\\src\\main\\java")//生成路径
.setFileOverride(true)//文件覆盖
.setIdType(IdType.AUTO) //主键策略
.setServiceName("%sService")//设置生成的service接口的名字的首字母是否为I 如IEmployeeService
.setBaseResultMap(true)//生成基本的结果集
.setBaseColumnList(true);//生成基本的列的集合 SQL片段供使用
```
---
#### 数据源配置
相关代码：
```
//2.数据源配置
DataSourceConfig dsConfig = new DataSourceConfig();
dsConfig.setDbType(DbType.MYSQL)//设置数据库类型 MYSQL ORACLE等
.setDriverName("com.mysql.jdbc.Driver")
.setUrl("jdbc:mysql://localhost:3306/mp-test")
.setUsername("root")
.setPassword("root");
```
---
#### 策略配置
相关代码：
```
//3.策略配置
StrategyConfig stConfig = new StrategyConfig();
stConfig.setCapitalMode(true)//全局大写命名
.setDbColumnUnderline(true)//指定表名 字段名是否使用下划线
.setNaming(NamingStrategy.underline_to_camel)//数据库表映射到实体的命名策略
.setTablePrefix("tbl_")//表名前缀
.setInclude("tbl_employee");//生成的表 可变参数
```
---
#### 包名策略配置
相关代码：
```
//4.包名策略配置
PackageConfig pkConfig = new PackageConfig();
pkConfig.setParent("top.tomxwd.mp")//设置父包，后面就不用写这段包名
.setMapper("mapper")
.setService("service")
.setController("controller")
.setEntity("beans")
.setXml("mapper");
```
---
#### 整合配置
相关代码：
```
//5.整合配置
AutoGenerator ag = new AutoGenerator();
ag.setGlobalConfig(config)
.setDataSource(dsConfig)
.setStrategy(stConfig)
.setPackageInfo(pkConfig);
```
---
#### 执行
相关代码：
```
//6.执行
ag.execute();
```
日志相关：
```
DEBUG 06-09 16:13:53,301 ================================================================= (ClassMap.java:91)
DEBUG 06-09 16:13:53,305 模板:/templates/entity.java.vm;  文件:E:\Java文件\mp04\src\main\java\top\tomxwd\mp\beans\Employee.java (VelocityTemplateEngine.java:72)
DEBUG 06-09 16:13:53,308 ResourceManager: found /templates/mapper.java.vm with loader org.apache.velocity.runtime.resource.loader.ClasspathResourceLoader (ResourceManagerImpl.java:439)
DEBUG 06-09 16:13:53,312 模板:/templates/mapper.java.vm;  文件:E:\Java文件\mp04\src\main\java\top\tomxwd\mp\mapper\EmployeeMapper.java (VelocityTemplateEngine.java:72)
DEBUG 06-09 16:13:53,317 ResourceManager: found /templates/mapper.xml.vm with loader org.apache.velocity.runtime.resource.loader.ClasspathResourceLoader (ResourceManagerImpl.java:439)
DEBUG 06-09 16:13:53,320 模板:/templates/mapper.xml.vm;  文件:E:\Java文件\mp04\src\main\java\top\tomxwd\mp\mapper\EmployeeMapper.xml (VelocityTemplateEngine.java:72)
DEBUG 06-09 16:13:53,321 ResourceManager: found /templates/service.java.vm with loader org.apache.velocity.runtime.resource.loader.ClasspathResourceLoader (ResourceManagerImpl.java:439)
DEBUG 06-09 16:13:53,324 模板:/templates/service.java.vm;  文件:E:\Java文件\mp04\src\main\java\top\tomxwd\mp\service\EmployeeService.java (VelocityTemplateEngine.java:72)
DEBUG 06-09 16:13:53,326 ResourceManager: found /templates/serviceImpl.java.vm with loader org.apache.velocity.runtime.resource.loader.ClasspathResourceLoader (ResourceManagerImpl.java:439)
DEBUG 06-09 16:13:53,327 模板:/templates/serviceImpl.java.vm;  文件:E:\Java文件\mp04\src\main\java\top\tomxwd\mp\service\impl\EmployeeServiceImpl.java (VelocityTemplateEngine.java:72)
DEBUG 06-09 16:13:53,329 ResourceManager: found /templates/controller.java.vm with loader org.apache.velocity.runtime.resource.loader.ClasspathResourceLoader (ResourceManagerImpl.java:439)
DEBUG 06-09 16:13:53,330 模板:/templates/controller.java.vm;  文件:E:\Java文件\mp04\src\main\java\top\tomxwd\mp\controller\EmployeeController.java (VelocityTemplateEngine.java:72)
DEBUG 06-09 16:13:53,396 ==========================文件生成完成！！！========================== (AutoGenerator.java:100)
```
注意：此时可能报错，因为pom中没有关于spring-webmvc相关的依赖。
<br>加入依赖：
```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>4.3.10.RELEASE</version>
</dependency>
```
在XxxServiceImpl中，不用再注入mapper了。<br>
原因：
1. 因为其继承了ServiceImpl，在ServiceImpl中已经完成了对Mapper对象的注入，直接在XxxServiceImpl中进行使用。
2. 在ServiceImpl中也帮我们提供了CRUD方法，基本的一些CRUD的方法在Service中不需要自己定义了。

---
## 总结：
比MGB好用。

---
## 总的生成代码
```
/**
	 * 代码生成 实例代码
	 * @throws Exception
	 */
	@Test
	public void testGenerator() throws Exception {
		//1.全局配置
		GlobalConfig config = new GlobalConfig();
		config.setActiveRecord(true)//是否支持AR模式
		.setAuthor("weidu.xie")//设置作者
		.setOutputDir("E:\\Java文件\\mp04\\src\\main\\java")//生成路径
		.setFileOverride(true)//文件覆盖
		.setIdType(IdType.AUTO) //主键策略
		.setServiceName("%sService")//设置生成的service接口的名字的首字母是否为I 如IEmployeeService
		.setBaseResultMap(true)//生成基本的结果集
		.setBaseColumnList(true);//生成基本的列的集合 SQL片段供使用
		//2.数据源配置
		DataSourceConfig dsConfig = new DataSourceConfig();
		dsConfig.setDbType(DbType.MYSQL)//设置数据库类型 MYSQL ORACLE等
		.setDriverName("com.mysql.jdbc.Driver")
		.setUrl("jdbc:mysql://localhost:3306/mp-test")
		.setUsername("root")
		.setPassword("root");
		//3.策略配置
		StrategyConfig stConfig = new StrategyConfig();
		stConfig.setCapitalMode(true)//全局大写命名
		.setDbColumnUnderline(true)//指定表名 字段名是否使用下划线
		.setNaming(NamingStrategy.underline_to_camel)//数据库表映射到实体的命名策略
		.setTablePrefix("tbl_")//表名前缀
		.setInclude("tbl_employee");//生成的表 可变参数
		//4.包名策略配置
		PackageConfig pkConfig = new PackageConfig();
		pkConfig.setParent("top.tomxwd.mp")//设置父包，后面就不用写这段包名
		.setMapper("mapper")
		.setService("service")
		.setController("controller")
		.setEntity("beans")
		.setXml("mapper");
		//5.整合配置
		AutoGenerator ag = new AutoGenerator();
		ag.setGlobalConfig(config)
		.setDataSource(dsConfig)
		.setStrategy(stConfig)
		.setPackageInfo(pkConfig);
		//6.执行
		ag.execute();
	}
```
