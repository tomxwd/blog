---
title: 04_Mybatis_接口式编程
date: 2019-08-09 12:57:43
tags: 
 - Java
 - Mybatis
categories:
 - Mybatis
---

# 04_Mybatis_接口式编程

由于用sqlSession来执行的话，对参数无法控制，若使用面向接口编程，就可以解决这个问题，即可以进行相关约束。

接口式编程的好处：

- 类型检查
- 明确返回值
- 接口本身是一种抽象，支持多种实现，不仅限于用mybatis来做，方便开发、扩展



## Mybatis配置文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://tomxwd.top:12389/mybatis"/>
                <property name="username" value="root"/>
                <property name="password" value="root"/>
            </dataSource>
        </environment>
    </environments>
    <!--将我们写好的sql映射文件注册到全局配置文件中来-->
    <mappers>
        <mapper resource="EmployeeMapper.xml"/>
    </mappers>
</configuration>
```



## log4j配置文件

```properties
# Global logging configuration
log4j.rootLogger=trace, stdout
# MyBatis logging configuration...
log4j.logger=TRACE
# Console output...
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%5p [%t] - %m%n
```



## Mapper接口

```java
public interface EmployeeMapper {

    Employee getEmpById(Integer id);

}
```



## Mapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!--
    namespace:命名空间,对应接口的全类名
    id：唯一标识符
    resultType：返回值类型
    #{id}:从传递过来的参数中取出id值
-->
    <mapper namespace="top.tomxwd.mybatis.mapper.EmployeeMapper">
    <select id="getEmpById" resultType="top.tomxwd.mybatis.bean.Employee">
        SELECT id,last_name lastName,email,gender FROM tbl_employee where id = #{id}
    </select>
</mapper>
```



## 测试

```java
private SqlSessionFactory factory;
private SqlSession sqlSession;

@Before
public void before() throws IOException {
    String resource = "mybatis-config.xml";
    InputStream stream = Resources.getResourceAsStream(resource);
    this.factory = new SqlSessionFactoryBuilder().build(stream);
    this.sqlSession = this.factory.openSession();
}

@After
public void after(){
    sqlSession.close();
}

@Test
public void test2() throws IOException{
    // 1. 获取接口的实现类对象
    // mybatis会为接口自动地创建代理对象，代理对象来执行增删改查方法
    EmployeeMapper mapper = sqlSession.getMapper(EmployeeMapper.class);
    Employee empById = mapper.getEmpById(1);
    System.out.println("empById = " + empById);
}
```

注意，这里的EmployeeMapper，如果进行打印的话，会发现是个proxy代理类型。 