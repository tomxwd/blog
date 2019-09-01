---
title: 28_MyBatis_映射文件_select_resultMap_自定义结果映射规则
date: 2019-08-11 10:30:30
tags: 
 - Java
 - Mybatis
categories:
 - Mybatis
---

# 28_MyBatis\_映射文件\_select_resultMap_自定义结果映射规则

## 自动映射（resultType）

1. 全局setting设置
   - **autoMappingBehavior默认是PARTIAL**，开启自动映射的功能，唯一的要求是列名和javaBean属性名一致。
   - 如果autoMappingBehavior设置为null则会取消自动映射
   - 数据库字段命名规范，POJO属性符合驼峰命名法，如A_COLUMN->aColumn，我们可以**开启自动驼峰命名规则功能，mapUnderscoreToCamelCase=true。**
2. 自定义resultMap，实现高级结果映射



## 自定义结果映射规则（resultMap）

resultMap跟resultType只能使用其中一个，且resultMap需要提前定义，定义封装的规则。

属性：

- type：自定义规则的java类型
- id：唯一id方便引用。

内部标签：

- `<id>`

  指定主键列的封装规则；

  id定义主键，底层会有优化。

  - column：数据库的列名
  - property：指定java属性名

- `<result>`

  定义普通列封装的规则。

  id也可以用result来指定，但是底层没优化。

  - column：数据库的列名
  - property：指定java属性名

**其他不指定规则的列，会进行自动封装。但是还是推荐所有的都写上去，方便代码阅读。**



### 接口类

```java
public interface EmployeeMapperPlus {

    Employee getEmpById(Integer id);

}
```



### sql映射文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="top.tomxwd.mybatis.mapper.EmployeeMapperPlus">

    <resultMap id="myEmp" type="top.tomxwd.mybatis.bean.Employee">
        <id column="id" property="id"></id>
        <result column="last_name" property="lastName"></result>
        <result column="email" property="email"></result>
        <result column="gender" property="gender"></result>
    </resultMap>
    
    <select id="getEmpById" resultMap="myEmp">
        select * from tbl_employee where id=#{id}
    </select>


</mapper>
```



### 测试

```java
public class MyBatisTest2 {

    private SqlSession sqlSession;
    private EmployeeMapperPlus mapper;

    @Before
    public void before() throws IOException {
        String resource = "mybatis-config.xml";
        InputStream stream = Resources.getResourceAsStream(resource);
        SqlSessionFactory build = new SqlSessionFactoryBuilder().build(stream);
        this.sqlSession = build.openSession();
        this.mapper = sqlSession.getMapper(EmployeeMapperPlus.class);
    }

    @After
    public void after(){
        sqlSession.commit();
        sqlSession.close();
    }
    
    @Test
    public void testSelect(){
        Employee empById = this.mapper.getEmpById(1);
        System.out.println("empById = " + empById);
    }

}
```

