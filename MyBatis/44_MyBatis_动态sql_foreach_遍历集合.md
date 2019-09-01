---
title: 44_MyBatis_动态sql_foreach_遍历集合
date: 2019-08-11 20:55:52
tags: 
 - Java
 - Mybatis
categories:
 - Mybatis
---

# 44_MyBatis\_动态sql_foreach_遍历集合

foreach标签

属性：

- collection：指定要遍历的集合，list类型的参数会特殊处理封装在map中，map的key就叫做list、collection，如果需要用自定义的，那么需要使用@Param来指定
- item：将遍历的元素赋值给指定的变量，然后用#{变量名}就可以取出；
- separator：就是用什么作为间隔符，比如in的时候用逗号（，）来做分隔符；
- close：用什么作为结尾，比如用in的时候最后要拼一个括号）；
- index：索引，遍历list的时候是索引，遍历map的时候是key；
- open：就是用什么作为开头，比如我们用in的时候要拼一个括号（；



## 接口类

```java
List<Employee> getEmpByIds(List<Integer> ids);
```

## sql映射文件

```xml
<select id="getEmpByIds" resultType="top.tomxwd.mybatis.bean.Employee">
    select * from tbl_employee where id in
    <foreach collection="list" item="item_id" separator="," close=")" open="(">
        #{item_id}
    </foreach>
</select>
```



## 测试

```java
@Test
public void testGetEmpByIds(){
    List<Employee> emps = empMapper.getEmpByIds(Arrays.asList(2,5));
    System.out.println("emps = " + emps);
}
```



## 输出结果

```
DEBUG [main] - ==>  Preparing: select * from tbl_employee where id in ( ? , ? ) 
DEBUG [main] - ==> Parameters: 2(Integer), 5(Integer)
TRACE [main] - <==    Columns: id, last_name, gender, email, d_id
TRACE [main] - <==        Row: 2, tim, 1, tim@tomxwd.top, 2
TRACE [main] - <==        Row: 5, tom, 1, 613@qq.com, 1
DEBUG [main] - <==      Total: 2
emps = [Employee(id=2, lastName=tim, email=tim@tomxwd.top, gender=1, dept=null), Employee(id=5, lastName=tom, email=613@qq.com, gender=1, dept=null)]
```

