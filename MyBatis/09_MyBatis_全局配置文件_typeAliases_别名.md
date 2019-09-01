---
title: 09_MyBatis_全局配置文件_typeAliases_别名
date: 2019-08-09 16:22:40
tags: 
 - Java
 - Mybatis
categories:
 - Mybatis
---

# 09_MyBatis\_全局配置文件\_typeAliases_别名

> [官网说明-别名](http://www.mybatis.org/mybatis-3/zh/configuration.html#typeAliases)

类型别名是为 Java 类型设置一个短的名字。 它只和 XML 配置有关，存在的意义仅在于用来减少类完全限定名的冗余。

**别名不区分大小写** ！

如果不指定具体的别名，那么就是类名，也可以直接指定包，起别名。



## 单个类起别名

```xml
<typeAliases>
    <typeAlias type="top.tomxwd.mybatis.bean.Employee" alias="emp"></typeAlias>
</typeAliases>
```



## 用包起别名

批量起别名，也是类的首字母小写。

```xml
<typeAliases>
    <package name="top.tomxwd.mybatis.bean"/>
</typeAliases>
```



## 别名冲突

如果在指定的包的子包下还有一个同名的类，这时候就要用注解了：

```java
@Alias("emp")
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

可见注解的优先级更高。



## 常见的java类型内建的对应类型别名

这是一些为常见的 Java 类型内建的相应的类型别名。它们都是不区分大小写的，注意对基本类型名称重复采取的特殊命名风格。

| 别名       | 映射的类型 |
| :--------- | :--------- |
| _byte      | byte       |
| _long      | long       |
| _short     | short      |
| _int       | int        |
| _integer   | int        |
| _double    | double     |
| _float     | float      |
| _boolean   | boolean    |
| string     | String     |
| byte       | Byte       |
| long       | Long       |
| short      | Short      |
| int        | Integer    |
| integer    | Integer    |
| double     | Double     |
| float      | Float      |
| boolean    | Boolean    |
| date       | Date       |
| decimal    | BigDecimal |
| bigdecimal | BigDecimal |
| object     | Object     |
| map        | Map        |
| hashmap    | HashMap    |
| list       | List       |
| arraylist  | ArrayList  |
| collection | Collection |
| iterator   | Iterator   |

如果不指定具体的别名，则是按照类名小写的形式