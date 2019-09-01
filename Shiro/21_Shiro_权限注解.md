---
title: 21_Shiro_权限注解
date: 2019-08-30 14:17:13
tags: 
 - Java
 - Shiro
categories:
 - Shiro
---

# 21_Shiro_权限注解

## 权限注解

| 注解                    | 用法                                                         |
| ----------------------- | ------------------------------------------------------------ |
| @RequiresAuthentication | 表示当前Subject已经通过login进行了身份验证，即**Subject.isAuthenticated()返回true；** |
| @RequiresUser           | 表示当前Subject已经**身份验证或者通过记住我登录的**；        |
| @RequiresGuest          | 表示当前Subject没有身份验证或通过记住我登录过，即是**游客身份**； |
| @RequiresRoles          | 比如@RequiresRoles(value={"admin","user"},logical=Logical.AND)：表示当前Subject**需要角色**admin和user； |
| @RequiresPermissions    | 比如@RequiresPermissions(value={"user:a","user:b"},logical=Logical.OR)：表示当前Subject**需要权限**user:a或user:b |



## 测试

### 准备工作：

1. 创建一个Service类用于测试，并注入spring容器中；

   ```java
   package top.tomxwd.shiro.service;
   
   import org.springframework.stereotype.Service;
   
   import java.util.Date;
   
   @Service
   public class ShiroService {
   
       public void testMethod(){
           System.out.println("testMethod,time :"+new Date());
       }
   
   }
   ```

2. 注入到Controller层，并编写映射方法；

   ```java
   @Controller
   @RequestMapping("/shiro")
   public class ShiroHandler {
   
       @Autowired
       private ShiroService service;
   
       @RequestMapping("/testShiroAnnotation")
       public String testShiroAnnotation(){
           service.testMethod();
           return "redirect:/list.jsp";
       }
   }
   ```

3. 在list.jsp下编写执行该映射方法的a标签；

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
   
   <a href="/shiro/testShiroAnnotation">Test ShiroAnnotation</a>
   
   <a href="shiro/logout">logout</a>
   </body>
   </html>
   ```





### 用上权限注解：

```java
@Service
public class ShiroService {

    @RequiresRoles(value = {"admin"})
    public void testMethod(){
        System.out.println("testMethod,time :"+new Date());
    }

}
```



再次用user进行访问该路径，发现报500错误，抛出org.apache.shiro.authz.UnauthorizedException该异常；这里可以用springmvc的异常处理来捕获，做统一处理；



这里需要注意的是，如果将权限注解打在了service层，会跟事务的注解发生冲突，因为两者都是用代理来做的，会发生类型转换异常；