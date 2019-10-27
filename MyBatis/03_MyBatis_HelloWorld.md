---
title: 03_Mybatis_HelloWorld
date: 2019-08-07 22:36:58
tags: 
 - Java
 - Mybatis
categories:
 - Mybatis
---

# 03_Mybatis_HelloWorld

> [参考文档](http://www.mybatis.org/mybatis-3)

## 导入依赖

```xml
<dependencies>
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
        <version>3.4.1</version>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>1.18.8</version>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>5.1.12</version>
    </dependency>
    <dependency>
        <groupId>log4j</groupId>
        <artifactId>log4j</artifactId>
        <version>1.2.17</version>
    </dependency>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
    </dependency>
</dependencies>
```

如果mapper.xml放在src/java目录下，编译不到target包的话加这段：

```xml
<resources>
    <resource>
        <directory>src/main/java</directory>
        <includes>
            <include>**/*.xml</include>
        </includes>
    </resource>
</resources>
```

## 创建数据库表

```sql
CREATE TABLE tbl_employee(
	id INT(11) PRIMARY KEY AUTO_INCREMENT,
  last_name VARCHAR(255),
	gender CHAR(1),
	email VARCHAR(255)
)
```

## 创建对应的javaBean类

```java
@Data
@AllArgsConstructor
@RequiredArgsConstructor
public class Employee {

    private Integer id;
    private String lastName;
    private String email;
    private String gender;

}
```

## 配置文件

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

改官网给的。

## 日志

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

改官网给的。

## Mapper文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!--
    namespace:命名空间
    id：唯一标识符
    resultType：返回值类型
    #{id}:从传递过来的参数中取出id值
-->
    <mapper namespace="top.tomxwd.mybatis.mapper.EmployeeMapper">
    <select id="selectEmp" resultType="top.tomxwd.mybatis.bean.Employee">
        SELECT id,last_name lastName,email,gender FROM tbl_employee where id = #{id}
    </select>
</mapper>
```

改官网给的。

## 测试

```java
package top.tomxwd.mybatis.test;

import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.Test;
import top.tomxwd.mybatis.bean.Employee;

import java.io.IOException;
import java.io.InputStream;

public class MyBatisTest {

    /**
     * 1. 根据xml配置文件（全局配置文件）创建一个SQLSessionFactory对象
     *    有数据源一些运行环境的信息
     * 2. sql映射文件，配置了每个SQL，以及SQL的封装规则等
     * 3. 将sql映射文件注册在全局配置文件中去。
     * 4. 写代码
     *      1）根据全局配置文件得到sqlSessionFactory
     *      2）使用sqlSession工厂，获得sqlSession对象，使用这个对象执行增删改查
     *          一个sqlSession就是代表和数据库的一次会话，用完需要关闭
     *      3）使用sql的唯一标识来告诉mybatis执行哪个sql，sql都是保存在sql映射文件中的
     * @throws IOException
     */
    @Test
    public void test() throws IOException {
        String resource = "mybatis-config.xml";
        InputStream stream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(stream);
        // 2. 获取sqlSession实例，能直接执行已经映射的sql语句
        // 两个参数
        // sql的唯一标识符
        // 执行sql要用的参数
        SqlSession sqlSession = sqlSessionFactory.openSession();
        Employee employee = (Employee) sqlSession.selectOne("top.tomxwd.mybatis.mapper.EmployeeMapper.selectEmp", 1);
        System.out.println("employee = " + employee);
        sqlSession.close();
    }


}
```

