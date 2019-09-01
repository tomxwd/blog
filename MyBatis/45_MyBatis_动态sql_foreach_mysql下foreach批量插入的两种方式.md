---
title: 45_MyBatis_动态sql_foreach_mysql下foreach批量插入的两种方式
date: 2019-08-11 21:41:06
tags: 
 - Java
 - Mybatis
categories:
 - Mybatis
---

# 45_MyBatis_动态sql_foreach_mysql下foreach批量插入的两种方式

## 实现要求

mysql支持的批量插入sql：

### 第一种方式

```sql
insert 
into 
tbl_employee 
(last_name,email,gender)
values
(Xxx,Xxx,Xxx),
(Xxx,Xxx,Xxx),
(Xxx,Xxx,Xxx)
```

### 第二种方式

```sql
insert into tbl_employee (last_name,email,gender) values (Xxx,Xxx,Xxx);
insert into tbl_employee (last_name,email,gender) values (Xxx,Xxx,Xxx);
insert into tbl_employee (last_name,email,gender) values (Xxx,Xxx,Xxx);
```



## 接口类

```java
Integer insertEmps(List<Employee> emps);
```



## sql映射文件

### 第一种方式

```xml
<insert id="insertEmps">
    insert into tbl_employee
    (last_name, email, gender)
    values
    <foreach collection="list" open="" close="" separator="," item="emp">
        (#{emp.lastName},#{emp.email},#{emp.gender})
    </foreach>
</insert>
```

### 第二种方式

```xml
<insert id="insertEmps">
    <foreach collection="list" item="emp" separator=";">
        insert into tbl_employee (last_name, email)
        values
        (#{emp.lastName},#{emp.email},#{emp.gender})
    </foreach>
</insert>
```

第二种方式发现运行出错了，但是查看sql语句并没有错，这里就需要注意了，这种批量插入的方式需要打开一个设置，**allowMultiQueries=true**，就是在url上面加就好了；

#### dbconfig.properties

```properties
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://Xxx:3306/mybatis?allowMultiQueries=true
jdbc.username=root
jdbc.password=root
```





## 测试

```java
@Test
public void testInsertEmps(){
    ArrayList<Employee> emps = new ArrayList<Employee>(5);
    for (int i = 0; i < 5; i++) {
        Employee emp = new Employee();
        emp.setEmail("123"+i+"@qq.com");
        emp.setLastName("tom"+i);
        emp.setGender(""+i%2);
        emps.add(emp);
    }
    Integer i = empMapper.insertEmps(emps);
    System.out.println("i = " + i);
}
```



## 结果

### 第一种方式

```
DEBUG [main] - ==>  Preparing: insert into tbl_employee (last_name, email, gender) values (?,?,?) , (?,?,?) , (?,?,?) , (?,?,?) , (?,?,?) 
DEBUG [main] - ==> Parameters: tom0(String), 1230@qq.com(String), 0(String), tom1(String), 1231@qq.com(String), 1(String), tom2(String), 1232@qq.com(String), 0(String), tom3(String), 1233@qq.com(String), 1(String), tom4(String), 1234@qq.com(String), 0(String)
DEBUG [main] - <==    Updates: 5
i = 5
```

### 第二种方式

```
DEBUG [main] - ==>  Preparing: insert into tbl_employee (last_name, email ,gender) values (?,?,?) ; insert into tbl_employee (last_name, email ,gender) values (?,?,?) ; insert into tbl_employee (last_name, email ,gender) values (?,?,?) ; insert into tbl_employee (last_name, email ,gender) values (?,?,?) ; insert into tbl_employee (last_name, email ,gender) values (?,?,?) 
DEBUG [main] - ==> Parameters: tom0(String), 1230@qq.com(String), 0(String), tom1(String), 1231@qq.com(String), 1(String), tom2(String), 1232@qq.com(String), 0(String), tom3(String), 1233@qq.com(String), 1(String), tom4(String), 1234@qq.com(String), 0(String)
DEBUG [main] - <==    Updates: 1
i = 1
```

