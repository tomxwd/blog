---
title: 32_MyBatis_映射文件_select_resultMap_关联查询_association分步查询
date: 2019-08-11 12:20:27
tags: 
 - Java
 - Mybatis
categories:
 - Mybatis
---

# 32_MyBatis\_映射文件\_select_resultMap\_关联查询_association分步查询

除了直接一条sql查出来，然后进行封装（级联或者association），我们还可以用association分布查询来实现。



使用association分布查询的封装规则：

- property属性，还是指向属性名；
- select属性，值为另一个mapper的方法，比如dept的getDeptById；
- column属性，也要指定具体的sql字段名；



## 实现例子

### 员工接口

```java
Employee2 getEmpAndDept3(Integer id);
```

### 部门接口

```java
Department getDeptById(Integer id);
```

### 员工SQL映射文件

```xml
<resultMap id="myEmpAndDept3" type="top.tomxwd.mybatis.bean.Employee2">
    <id column="id" property="id"></id>
    <result column="last_name" property="lastName"></result>
    <result column="email" property="email"></result>
    <result column="gender" property="gender"></result>
    <association property="dept" select="top.tomxwd.mybatis.mapper.DepartmentMapper.getDeptById" column="d_id"></association>
</resultMap>

<select id="getEmpAndDept3" resultMap="myEmpAndDept3">
    select * from tbl_employee where id = #{id}
</select>
```

### 部门SQL映射文件

```xml
<select id="getDeptById" resultType="top.tomxwd.mybatis.bean.Department">
    select id,department_name departmentName from tbl_department where id = #{id}
</select>
```

### 测试

```java
public class MyBatisTerst3 {

    private SqlSession sqlSession;
    private EmployeeMapperPlus empMapper;
    private DepartmentMapper deptMapper;

    @Before
    public void before() throws IOException {
        String resource = "mybatis-config.xml";
        InputStream stream = Resources.getResourceAsStream(resource);
        SqlSessionFactory build = new SqlSessionFactoryBuilder().build(stream);
        this.sqlSession = build.openSession();
        this.empMapper = sqlSession.getMapper(EmployeeMapperPlus.class);
        this.deptMapper = sqlSession.getMapper(DepartmentMapper.class);
    }

    @After
    public void after(){
        sqlSession.commit();
        sqlSession.close();
    }

    @Test
    public void testGetEmpAndDept(){
        Employee2 empAndDept3 = this.empMapper.getEmpAndDept3(1);
        System.out.println("empAndDept3 = " + empAndDept3);
    }

}
```

### 控制台打印结果

```
DEBUG [main] - ==>  Preparing: select * from tbl_employee where id = ? 
DEBUG [main] - ==> Parameters: 1(Integer)
TRACE [main] - <==    Columns: id, last_name, gender, email, d_id
TRACE [main] - <==        Row: 1, tom, 0, tom@tomxwd.top, 1
DEBUG [main] - ====>  Preparing: select id,department_name departmentName from tbl_department where id = ? 
DEBUG [main] - ====> Parameters: 1(Integer)
TRACE [main] - <====    Columns: id, departmentName
TRACE [main] - <====        Row: 1, 部门1
DEBUG [main] - <====      Total: 1
DEBUG [main] - <==      Total: 1
empAndDept3 = Employee2(id=1, lastName=tom, email=tom@tomxwd.top, gender=0, dept=Department(id=1, departmentName=部门1))
```

其实就是发了两个sql语句，再封装到对象里，这样sql就可以不用自己去写太多。