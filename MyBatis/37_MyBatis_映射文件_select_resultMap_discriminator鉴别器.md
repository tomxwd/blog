---
title: 37_MyBatis_映射文件_select_resultMap_discriminator鉴别器
date: 2019-08-11 15:42:00
tags: 
 - Java
 - Mybatis
categories:
 - Mybatis
---

# 37_MyBatis\_映射文件_select_resultMap_discriminator鉴别器

discriminator是resultMap的一个子标签，是鉴别器；

**mybatis可以使用discriminator鉴别器判断某列的值，然后根据某列的值改变封装行为；**

例如性别：封装Employee的时候：

如果查询是女生，则将部门信息查询出来，否则不查询；

如果查询是男生，则把last_name这一列的值赋给email；



## discriminator用法

属性：

- column：指定判定的列名
- javaType：列值对应的java类型

### 子标签case

属性：

- value：列的属性值为什么的时候，生效
- resultType：指定封装的结果类型，意思就是说，在封装什么对象的时候生效，影响什么对象的封装。【必填】；
- resultMap：同上；



## 例子实现

### 接口类

```java
Employee2 getEmpById2(Integer id);
```



### SQL映射文件

```xml
<resultMap id="myEmp2" type="top.tomxwd.mybatis.bean.Employee2">
    <id column="id" property="id"></id>
    <result column="last_name" property="lastName"></result>
    <result column="email" property="email"></result>
    <result column="gender" property="gender"></result>
    <discriminator javaType="string" column="gender" >
        <!-- 女生 -->
        <case value="0" resultType="top.tomxwd.mybatis.bean.Employee2">
            <association property="dept" select="top.tomxwd.mybatis.mapper.DepartmentMapper.getDeptById" column="d_id"></association>
        </case>
        <!-- 男生 -->
        <case value="1" resultType="top.tomxwd.mybatis.bean.Employee2">
            <id column="id" property="id"></id>
            <result column="last_name" property="lastName"></result>
            <result column="last_name" property="email"></result>
            <result column="gender" property="gender"></result>
        </case>
    </discriminator>
</resultMap>

<select id="getEmpById2" resultMap="myEmp2">
    select * from tbl_employee where id=#{id}
</select>
```

主要关注resultMap；



### 测试

```java
    @Test
    public void testGetEmp2(){
        Employee2 girl = this.empMapper.getEmpById2(1);
        System.out.println("girl = " + girl);
        System.out.println("------------------");
        Employee2 boy = this.empMapper.getEmpById2(2);
        System.out.println("boy = " + boy);
    }
```



### 控制台输出

```
DEBUG [main] - ==>  Preparing: select * from tbl_employee where id=? 
DEBUG [main] - ==> Parameters: 1(Integer)
TRACE [main] - <==    Columns: id, last_name, gender, email, d_id
TRACE [main] - <==        Row: 1, tom, 0, tom@tomxwd.top, 1
DEBUG [main] - <==      Total: 1
DEBUG [main] - ==>  Preparing: select id,department_name departmentName from tbl_department where id = ? 
DEBUG [main] - ==> Parameters: 1(Integer)
TRACE [main] - <==    Columns: id, departmentName
TRACE [main] - <==        Row: 1, 部门1
DEBUG [main] - <==      Total: 1
girl = Employee2(id=1, lastName=tom, email=tom@tomxwd.top, gender=0, dept=Department(id=1, departmentName=部门1, emps=null))
------------------
DEBUG [main] - ==>  Preparing: select * from tbl_employee where id=? 
DEBUG [main] - ==> Parameters: 2(Integer)
TRACE [main] - <==    Columns: id, last_name, gender, email, d_id
TRACE [main] - <==        Row: 2, tim, 1, tim@tomxwd.top, 2
DEBUG [main] - <==      Total: 1
boy = Employee2(id=2, lastName=tim, email=tim, gender=1, dept=null)
DEBUG [main] - Resetting autocommit to true on JDBC Connection [com.mysql.jdbc.JDBC4Connection@4923ab24]
DEBUG [main] - Closing JDBC Connection [com.mysql.jdbc.JDBC4Connection@4923ab24]
DEBUG [main] - Returned connection 1227074340 to pool.
```

可以看到结果，女生的时候是会去查部门，而男生不查部门，而且把email的值替换成了last_name的值了；