---
title: 29_MyBatis_映射文件_select_resultMap_关联查询_环境搭建
date: 2019-08-11 10:54:20
tags: 
 - Java
 - Mybatis
categories:
 - Mybatis
---

# 29_MyBatis\_映射文件\_select_resultMap\_关联查询_环境搭建

resultMap还有更加强大的功能——关联查询。

比如一个Employee对应一个Department信息，即查出员工的时候同时查出部门信息。

## 表信息

tbl_department:

```sql
CREATE TABLE `tbl_department` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `department_name` varchar(255) COLLATE utf8mb4_unicode_ci DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

tbl_employee:

```sql
CREATE TABLE `tbl_employee` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `last_name` varchar(255) COLLATE utf8mb4_unicode_ci DEFAULT NULL,
  `gender` char(1) COLLATE utf8mb4_unicode_ci DEFAULT NULL,
  `email` varchar(255) COLLATE utf8mb4_unicode_ci DEFAULT NULL,
  `d_id` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=6 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```



## javaBean

Employee2:

```java
@Data
@NoArgsConstructor
@RequiredArgsConstructor
public class Employee2 {

    private Integer id;
    private String lastName;
    private String email;
    private String gender;
    private Department dept;

}
```

Department:

```java
@Data
@RequiredArgsConstructor
@NoArgsConstructor
public class Department {

    private Integer id;
    private String departmentName;

}
```