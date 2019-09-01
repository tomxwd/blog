---
title: 39_MyBatis_动态sql_if_判断&OGNL
date: 2019-08-11 16:56:32
tags: 
 - Java
 - Mybatis
categories:
 - Mybatis
---

# 39_MyBatis\_动态sql_if_判断&OGNL

## OGNL

> [OGNL官网](http://commons.apache.org/proper/commons-ognl/)

**OGNL（Object Graph Navigation Language）对象图导航语言，这是一种强大的表达式语言，通过它可以非常方便的来操作对象的属性，类似于我们的EL，SpEL等。**

- 访问对象属性

  - person.name

- 调用方法

  - person.getName()

- 调用静态属性/方法

  - @java.lang.Math@PI
  - @java.util.UUID@randomUUID()

- 调用构造方法

  - new top.tomxwd.bean.Employee('admin').name

- 运算符

  - +,-,*,/,%

- 逻辑运算符：

  - int,not,in,>,<,<=,>=,==,!=（***注意xml中有些特殊符号需要转义**）

- 访问集合伪属性：

  | 类型           | 伪属性        | 伪属性对应的Java方法                       |
  | -------------- | ------------- | ------------------------------------------ |
  | List、Set、Map | size、isEmpty | List/Set/Map.size(),List/Set/Map.isEmpty() |
  | List、Set      | iterator      | List.iterator()、Set.iterator()            |
  | Map            | keys、values  | Map.keySet()、Map.values()                 |
  | Iterator       | next、hasNext | Iterator.next()、Iterator.hasNext()        |

  

## if场景

查询员工，要求，携带了哪个字段，查询条件就带上这个字段的值去查询；

属性：

- test：判断表达式（用的是OGNL表达式）



### 接口类

```java
public interface EmployeeMapper {

    List<Employee> getEmpsByConditionIf(Employee employee);

}
```



### sql映射文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

    <mapper namespace="top.tomxwd.mybatis.mapper.EmployeeMapper">

    <select id="getEmpsByConditionIf" resultType="top.tomxwd.mybatis.bean.Employee">
        select * from tbl_employee
        WHERE
        <if test="id!=null">
            id=#{id}
        </if>
        <if test="lastName!=null and lastName!=''">
            and last_name like "%"#{lastName}"%"
        </if>
        <if test="email!=null and email.trim()!=''">
            and email=#{email}
        </if>
        <if test="gender==0 or gender==1">
            and gender=#{gender}
        </if>
    </select>


</mapper>
```



### 测试

```java
public class MyBatisTest {

    private SqlSession sqlSession;
    private EmployeeMapper empMapper;

    @Before
    public void before() throws IOException {
        String resource = "mybatis-config.xml";
        InputStream stream = Resources.getResourceAsStream(resource);
        SqlSessionFactory build = new SqlSessionFactoryBuilder().build(stream);
        this.sqlSession = build.openSession();
        this.empMapper = sqlSession.getMapper(EmployeeMapper.class);
    }

    @After
    public void after(){
        sqlSession.commit();
        sqlSession.close();
    }
    @Test
    public void testIf(){
        Employee emp = new Employee();
        emp.setLastName("t");
        emp.setEmail("613@qq.com");
        List<Employee> emps = empMapper.getEmpsByConditionIf(emp);
        System.out.println("emps = " + emps);
    }

}
```



### 结果

```
DEBUG [main] - ==>  Preparing: select * from tbl_employee WHERE last_name like "%"?"%" and email=? 
DEBUG [main] - ==> Parameters: t(String), 613@qq.com(String)
TRACE [main] - <==    Columns: id, last_name, gender, email, d_id
TRACE [main] - <==        Row: 5, tom, 1, 613@qq.com, 1
DEBUG [main] - <==      Total: 1
emps = [Employee(id=5, lastName=tom, email=613@qq.com, gender=1, dept=null)]
```

可以根据输出的sql看到，拼接的情况。