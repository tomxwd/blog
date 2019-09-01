---
title: 20_Shiro_标签
date: 2019-08-30 13:54:47
tags: 
 - Java
 - Shiro
categories:
 - Shiro
---

# 20_Shiro_标签

## Shiro标签

Shiro提供了JSTL标签库用于在JSP页面进行权限控制，如根据登录用户显示相应的页面按钮；

- guest标签：用户没有身份验证时显示相应的信息，即游客访问信息：

  ```jsp
  <shiro:guest>
  	欢迎游客访问，<a href="login.jsp">登录</a>
  </shiro:guest>
  ```

- user标签：用户已经经过认证/**记住我**登录后显示相应的信息：

  ```jsp
  <shiro:user>
  	欢迎[<shiro:principal/>]登录，<a href="logout">退出</a>
  </shiro:user>
  ```

- authenticated标签：用户已经身份验证通过，即Subject.login登录成功，**不是记住我登录的**：

  ```jsp
  <shiro:authenticated>
  	用户[<shiro:principal/>]已身份验证通过
  </shiro:authenticated>
  ```

- notAuthenticated标签：用户未进行身份验证，即没有调用Subject.login进行登录，**包括记住我自动登录**的也属于未进行身份验证：

  ```jsp
  <shiro:notAuthenticated>
  	未身份验证（包括记住我）
  </shiro:notAuthenticated>
  ```

- pincipal标签：**显示用户身份信息**，默认调用Subject.getPrincipal()获取，即Primary Principal。

  ```jsp
  <shiro:principal property="username" />
  ```

- hasRole标签：如果当前Subjcet有角色将显示body体内容：

  ```jsp
  <shiro:hasRole name="admin">
  	用户[<shiro:principal/>]拥有角色admin
  </shiro:hasRole>
  ```

- hasAnyRoles标签：如果当前Subject有任意一个人角色（或的关系）将显示body体内容：

  ```jsp
  <shiro:hasAnyRoles name="admin,user">
  	用户[<shiro:principal/>]拥有角色admin或user
  </shiro:hasAnyRoles>
  ```

- lacksRole：如果当前Subject没有角色将显示body的内容：

  ```jsp
  <shiro:lacksRole name="admin">
  	用户[<shiro:principal/>]没有角色admin
  </shiro:lacksRole>
  ```

- hasPermission：如果当前Subject有权限将显示body体内容：

  ```jsp
  <shiro:hasPermission name="user:create">
  	用户[<shiro:principal/>]拥有权限user:create
  </shiro:hasPermission>
  ```

- lacksPermission：如果当前Subject没有权限将显示body体内容：

  ```jsp
  <shiro:lacksPermission name="org:create">
  	用户[<shiro:principal/>]没有权限org:create
  </shiro:lacksPermission>
  ```

  

## 使用Shiro标签

要使用shiro标签，需要引入标签库：

```jsp
<%@ taglib prefix="shiro" uri="http://shiro.apache.org/tags"%>
```

list.jsp：

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@ taglib prefix="shiro" uri="http://shiro.apache.org/tags"%>
<html>
<head>
    <title>List</title>
</head>
<body>
<h4>List Page</h4><br><br>

Welcome:<shiro:principal></shiro:principal><br><br>

<shiro:hasRole name="admin">
    <a href="admin.jsp">adminPage</a><br><br>
</shiro:hasRole>
<shiro:hasRole name="user">
    <a href="user.jsp">userPage</a><br><br>
</shiro:hasRole>

<a href="shiro/logout">logout</a>
</body>
</html>
```



