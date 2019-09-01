---
title: 16_MyBatis_映射文件_增删改查
date: 2019-08-09 19:31:19
tags: 
 - Java
 - Mybatis
categories:
 - Mybatis
---

# 16_MyBatis\_映射文件_增删改查

对mybatis的增删改查操作进行简单测试；

**记住：**

- sqlSession需要手动提交事务！sqlSession.commit();
  - 要么就用sqlSession.commit(true);开启自动提交，即可。

- 删改查是可以返回Integer、Long、Boolean数据类型的，只不过一个是返回影响的条数，一个是返回是否成功标志。



## dbconfig.properties

```properties
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://XXX:3306/mybatis
jdbc.username=root
jdbc.password=root
```



## log4j.properties

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



## mybatis-config.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    


    <properties resource="dbconfig.properties"></properties>
    
    <settings>
        <!--开启驼峰命名-->
        <setting name="mapUnderscoreToCamelCase" value="true"/>
    </settings>

    <typeAliases>
        <package name="top.tomxwd.mybatis.bean"/>
    </typeAliases>

    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="${jdbc.driver}"/>
                <property name="url" value="${jdbc.url}"/>
                <property name="username" value="${jdbc.username}"/>
                <property name="password" value="${jdbc.password}"/>
            </dataSource>
        </environment>
    </environments>

    <mappers>
        <mapper resource="EmployeeMapper.xml"/>
    </mappers>
</configuration>
```



## Employee

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



## EmployeeMapper接口

```java
public interface EmployeeMapper {

    Employee getEmpById(Integer id);

    Integer addEmp(Employee employee);

    Integer updateEmp(Employee employee);

    Integer deleteEmp(Integer id);

}
```



## EmployeeMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="top.tomxwd.mybatis.mapper.EmployeeMapper">

    <select id="getEmpById" resultType="Employee">
        select * from tbl_employee where id=#{id}
    </select>

    <!--paramterType可以省略，要写就写全名-->
    <insert id="addEmp" parameterType="top.tomxwd.mybatis.bean.Employee">
        insert into tbl_employee (last_name,email,gender) values (#{lastName},#{email},#{gender})
    </insert>

    <update id="updateEmp">
        update tbl_employee set last_name=#{lastName},email=#{email},gender=#{gender} where id=#{id}
    </update>

    <delete id="deleteEmp">
        delete from tbl_employee where id=#{id}
    </delete>


</mapper>
```





## 测试

```java
public class MyBatisTest {

    private SqlSession sqlSession;
    private EmployeeMapper mapper;

    @Before
    public void before() throws IOException {
        String resource = "mybatis-config.xml";
        InputStream stream = Resources.getResourceAsStream(resource);
        SqlSessionFactory build = new SqlSessionFactoryBuilder().build(stream);
        this.sqlSession = build.openSession();
        this.mapper = sqlSession.getMapper(EmployeeMapper.class);
    }

    @After
    public void after(){
        sqlSession.commit();
        sqlSession.close();
    }

    @Test
    public void testSelect(){
        Employee empById = mapper.getEmpById(3);
        System.out.println("emp = " + empById);
    }

    @Test
    public void testAddEmp(){
        Integer i = mapper.addEmp(new Employee(null, "tom", "613@qq.com", "1"));
        System.out.println("i = " + i);
    }
    @Test
    public void testUpdateEmp(){
        Employee employee = new Employee(4, "ttt", "487@qq.com", "0");
        Integer integer = mapper.updateEmp(employee);
        System.out.println("integer = " + integer);
    }

    @Test
    public void testDeleteEmp(){
        Integer integer = mapper.deleteEmp(3);
        System.out.println("integer = " + integer);
    }

}
```

