---
title: 09_Shiro_实现认证Realm以及登出logout功能
date: 2019-08-30 08:07:48
tags: 
 - Java
 - Shiro
categories:
 - Shiro
---

# 09_Shiro_实现认证Realm以及登出logout功能

## 实现认证Realm

1. 把AuthenticationToken转换为UsernamePasswordToken；
2. 从UsernamePasswordToken中来获取username；
3. 调用数据库的方法，从数据库中查询用户的记录；
4. 若用户不存在则可以抛出异常（UnknownAccountException）；
5. 根据用户信息的情况，决定是否需要抛出其他的AuthenticationException异常，锁定等情况；
6. 根据用户情况，来构建AuthenticationInfo对象并返回，，通常使用的实现类为SimpleAuthenticationInfo；
   - 以下信息是从数据库中获取的：
   - principal：认证的实体信息，可以是username，也可以是数据表对应的实体类对象
   - credentials：密码
   - realmName：当前realm对象的name，调用父类的getName方法即可

实现：

```java
public class ShiroRealm extends AuthenticatingRealm {
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
        // 1. 把AuthenticationToken转换为UsernamePasswordToken
        UsernamePasswordToken token = ((UsernamePasswordToken) authenticationToken);
        // 2. 从UsernamePasswordToken中来获取username
        String username = token.getUsername();
        // 3. 调用数据库的方法，从数据库中查询用户的记录
        System.out.println("从数据库中获取username："+username+" 所对应的用户信息。");

        // 4. 若用户不存在则可以抛出异常（UnknownAccountException）
        if("unknown".equals(username)){
            throw new UnknownAccountException("用户不存在");
        }
        // 5. 根据用户信息的情况，决定是否需要抛出其他的AuthenticationException异常，锁定等情况
        if("monster".equals(username)){
            throw new LockedAccountException("用户被锁定");
        }
        // 6. 根据用户情况，来构建AuthenticationInfo对象并返回，通常使用的实现类为SimpleAuthenticationInfo
        // 以下信息是从数据库中获取的
        // 1) principal：认证的实体信息，可以是username，也可以是数据表对应的实体类对象
        Object principal = username;
        // 2) credentials：密码
        Object credentials="123456";
        // 3) realmName：当前realm对象的name，调用父类的getName方法即可
        String realmName = getName();
        SimpleAuthenticationInfo simpleAuthenticationInfo = new SimpleAuthenticationInfo(principal, credentials, realmName);
        return simpleAuthenticationInfo;
    }
}
```

运行结果：

```
从数据库中获取username：1111 所对应的用户信息。
登录失败：Submitted credentials for token [org.apache.shiro.authc.UsernamePasswordToken - 1111, rememberMe=true] did not match the expected credentials.
从数据库中获取username：unknown 所对应的用户信息。
登录失败：用户不存在
从数据库中获取username：monster 所对应的用户信息。
登录失败：用户被锁定
从数据库中获取username：1111 所对应的用户信息。
```



## logout登出

在list.jsp中添加语句：

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>List</title>
</head>
<body>
<h4>List Page</h4>
<a href="shiro/logout">logout</a>
</body>
</html>
```

配置的URL是shiro/logout，但是不用我们自己去实现，直接在spring-shiro配置文件中进行配置：

spring-shiro.xml：

```xml
<!--
   配置哪些页面需要受保护，
   以及访问这些页面需要的权限
   1） anon：可以被匿名访问
   2） authc：必须认证（即登录）后才可以访问的页面
   3） logout: 登出
 -->
<property name="filterChainDefinitions">
    <value>
        /login.jsp = anon
        /shiro/login = anon
        /shiro/logout = logout
        # everything else requires authentication:
        /** = authc
    </value>
</property>
```

