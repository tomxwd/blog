---
title: 16_Shiro_权限配置
date: 2019-08-30 11:00:53
tags: 
 - Java
 - Shiro
categories:
 - Shiro
---

# 16_Shiro_权限配置

![授权](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/Shiro/16%E6%8E%88%E6%9D%83.png)

![授权方式](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/Shiro/16%E6%8E%88%E6%9D%83%E6%96%B9%E5%BC%8F.png)

![默认拦截器](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/Shiro/16%E9%BB%98%E8%AE%A4%E6%8B%A6%E6%88%AA%E5%99%A8.png)

![身份验证相关的](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/Shiro/16%E8%BA%AB%E4%BB%BD%E9%AA%8C%E8%AF%81%E7%9B%B8%E5%85%B3%E7%9A%84.png)

![授权相关的](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/Shiro/16%E6%8E%88%E6%9D%83%E7%9B%B8%E5%85%B3%E7%9A%84.png)

![其他](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/Shiro/16%E5%85%B6%E4%BB%96.png)



![Permissions](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/Shiro/16Permissions.png)

![Shiro的Permissions](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/Shiro/16Shiro%E7%9A%84Permissions.png)



## 测试

准备工作：

list.jsp页面：

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>List</title>
</head>
<body>
<h4>List Page</h4><br><br>
<a href="admin.jsp">adminPage</a><br><br>
<a href="user.jsp">userPage</a><br><br>
<a href="shiro/logout">logout</a>
</body>
</html>
```

再复制出一份admin.jsp以及user.jsp即可；



配置spring-shiro.xml 过滤角色：

```xml
<bean id="shiroFilter" class="org.apache.shiro.spring.web.ShiroFilterFactoryBean">
    <property name="securityManager" ref="securityManager"/>
    <property name="loginUrl" value="/login.jsp"/>
    <property name="successUrl" value="/list.jsp"/>
    <property name="unauthorizedUrl" value="/unauthorized.jsp"/>
    <!--
        配置哪些页面需要受保护，
        以及访问这些页面需要的权限
        1） anon：可以被匿名访问
        2） authc：必须认证（即登录）后才可以访问的页面
        3） logout: 登出
        4） roles：角色过滤器
    -->
    <property name="filterChainDefinitions">
        <value>
            /login.jsp = anon
            /shiro/login = anon
            /shiro/logout = logout
            
            /user.jsp = roles[user]
            /admin.jsp = roles[admin]

            # everything else requires authentication:
            /** = authc
        </value>
    </property>
</bean>
```

注意：到这时候就算运行项目也看不到效果的，只会显示user.jsp和admin.jsp无权限，因为还没配角色等信息。





