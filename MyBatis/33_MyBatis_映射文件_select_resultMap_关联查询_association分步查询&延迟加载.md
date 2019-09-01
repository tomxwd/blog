---
title: 33_MyBatis_映射文件_select_resultMap_关联查询_association分步查询&延迟加载
date: 2019-08-11 12:20:27
tags: 
 - Java
 - Mybatis
categories:
 - Mybatis
---

# 33_MyBatis\_映射文件\_select_resultMap\_关联查询_association分步查询&延迟加载

分步查询还支持延迟加载，也就是平常所说的懒加载；

我们现在通过员工查询部门的时候，是直接一起查询出来的，但是我们希望部门信息是在我们使用到的时候再去查询。

**我们只要在分步查询的基础之上加上两个配置，就可以实现延迟加载**

| 设置名                | 描述                                                         | 有效值        | 默认值                                       |
| :-------------------- | :----------------------------------------------------------- | :------------ | :------------------------------------------- |
| lazyLoadingEnabled    | 延迟加载的全局开关。当开启时，所有关联对象都会延迟加载。 特定关联关系中可通过设置 `fetchType`属性来覆盖该项的开关状态。 | true \| false | false                                        |
| aggressiveLazyLoading | 当开启时，任何方法的调用都会加载该对象的所有属性。 否则，每个属性会按需加载（参考 `lazyLoadTriggerMethods`)。 | true \| false | false （在 3.4.1 及之前的版本默认值为 true） |

1. **全局配置文件的setting属性，把lazyLoadingEnabled设置为true；**
2. **全局配置文件的setting属性，把aggressiveLazyLoading设置为false；**

## 例子实现

### mybatis全局配置文件

```xml
<settings>
    <setting name="lazyLoadingEnabled" value="true"/>
    <setting name="aggressiveLazyLoading" value="false"/>
</settings>
```

### 测试类

```java
@Test
public void testGetEmpAndDept(){
    Employee2 empAndDept3 = this.empMapper.getEmpAndDept3(1);
    System.out.println("empAndDept3.getLastName() = " + empAndDept3.getLastName());
    System.out.println("------------------------------");
    System.out.println("empAndDept3.getDept() = " + empAndDept3.getDept());
}
```

### 测试结果

```
DEBUG [main] - ==>  Preparing: select * from tbl_employee where id = ? 
DEBUG [main] - ==> Parameters: 1(Integer)
TRACE [main] - <==    Columns: id, last_name, gender, email, d_id
TRACE [main] - <==        Row: 1, tom, 0, tom@tomxwd.top, 1
DEBUG [main] - <==      Total: 1
empAndDept3.getLastName() = tom
------------------------------
DEBUG [main] - ==>  Preparing: select id,department_name departmentName from tbl_department where id = ? 
DEBUG [main] - ==> Parameters: 1(Integer)
TRACE [main] - <==    Columns: id, departmentName
TRACE [main] - <==        Row: 1, 部门1
DEBUG [main] - <==      Total: 1
empAndDept3.getDept() = Department(id=1, departmentName=部门1)
```

可以看到，只有当我们需要部门信息的时候，才会去发sql语句查询。