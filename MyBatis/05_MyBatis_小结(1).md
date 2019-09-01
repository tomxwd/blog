---
title: 05_MyBatis_小结(1)
date: 2019-08-09 15:30:09
tags: 
 - Java
 - Mybatis
categories:
 - Mybatis
---

# 05_MyBatis_小结(1)

首先我们使用了以前mybatis的用法，没有用接口，比较麻烦，需要自己去确定入参以及返回的类型，属于老做法，现在都是使用接口式进行编程，所以我们也使用了接口式方法进行尝试。

1. 接口式编程
   - 原生：		Dao 		=====>			DaoImpl
   - mybatis： Mapper   =====>            XxxMapper.xml
2. SqlSession代表和数据库的一次会话，**用完需要关闭**；
3. SqlSession和Connection一样，都是**非线程安全的**，所以并不可以将SqlSession作为成员变量进行使用，在之前的测试文件中，只是为了方便测试。每次使用都应该去获取新的对象。
4. mapper接口没有实现类，但是mybatis会为接口生成一个proxy代理对象，也就是说我们最终用`EmployeeMapper empMapper = sqlSession.getMapper(EmployeeMapper.class);`是通过接口和xml文件进行绑定后的结果。
5. 两个重要的配置文件：
   - mybatis全局配置文件：包含了数据库连接池信息，事务管理器信息等。。。系统运行环境信息**（这个可以用xml文件的形式，也可以用java代码，来将信息塞入到factory对象中，使配置生效）**
   - Sql映射文件：保存了每一个sql语句的映射信息（这个映射文件将sql抽取出来操作，灵活性高，开发人员自己写sql）