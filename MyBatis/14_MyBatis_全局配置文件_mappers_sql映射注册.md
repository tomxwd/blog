---
title: 14_MyBatis_全局配置文件_mappers_sql映射注册
date: 2019-08-09 19:15:26
tags: 
 - Java
 - Mybatis
categories:
 - Mybatis
---

# 14_MyBatis\_全局配置文件\_mappers_sql映射注册

> [官网——mappers](http://www.mybatis.org/mybatis-3/zh/configuration.html#mappers)

## 官网文档说法

既然 MyBatis 的行为已经由上述元素配置完了，我们现在就要定义 SQL 映射语句了。 但是首先我们需要告诉 MyBatis 到哪里去找到这些语句。 Java 在自动查找这方面没有提供一个很好的方法，所以最佳的方式是告诉 MyBatis 到哪里去找映射文件。 你可以使用相对于类路径的资源引用， 或完全限定资源定位符（包括 `file:///` 的 URL），或类名和包名等。例如：

```xml
<!-- 使用相对于类路径的资源引用 -->
<mappers>
  <mapper resource="org/mybatis/builder/AuthorMapper.xml"/>
  <mapper resource="org/mybatis/builder/BlogMapper.xml"/>
  <mapper resource="org/mybatis/builder/PostMapper.xml"/>
</mappers>
<!-- 使用完全限定资源定位符（URL） -->
<mappers>
  <mapper url="file:///var/mappers/AuthorMapper.xml"/>
  <mapper url="file:///var/mappers/BlogMapper.xml"/>
  <mapper url="file:///var/mappers/PostMapper.xml"/>
</mappers>
<!-- 使用映射器接口实现类的完全限定类名 -->
<mappers>
  <mapper class="org.mybatis.builder.AuthorMapper"/>
  <mapper class="org.mybatis.builder.BlogMapper"/>
  <mapper class="org.mybatis.builder.PostMapper"/>
</mappers>
<!-- 将包内的映射器接口实现全部注册为映射器 -->
<mappers>
  <package name="org.mybatis.builder"/>
</mappers>
```

这些配置会告诉了 MyBatis 去哪里找映射文件，剩下的细节就应该是每个 SQL 映射文件了，也就是接下来我们要讨论的。

## 具体

- resource就是类路径下的资源；
- url就是网络或者磁盘下的资源；
- 而class则需要遵循一个规则，即**接口和映射文件需要放在同一个包下，且同名**才可以生效。或者用**注解形式的接口**。
- `<package>`可以进行批量注册，规则跟class一样。

## 提议

比较重要的还是用sql映射文件来写，不重要的接口为了开发快速，可以使用注解形式来做。