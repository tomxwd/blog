---
title: 08_Shiro_实现认证流程
date: 2019-08-29 17:03:03
tags: 
 - Java
 - Shiro
categories:
 - Shiro
---

# 08_Shiro_实现认证流程

1. 创建表单，在login.jsp中：

   ```jsp
   <form action="shiro/login" method="POST">
       username:<input type="text" name="username"><br><br>
       password:<input type="password" name="password"><br><br>
       <input type="submit" value="Submit">
   </form>
   ```

2. 创建一个请求处理器ShiroController：

   ```java
   package top.tomxwd.controller;
   
   import org.apache.shiro.SecurityUtils;
   import org.apache.shiro.authc.*;
   import org.apache.shiro.subject.Subject;
   import org.springframework.stereotype.Controller;
   import org.springframework.web.bind.annotation.RequestMapping;
   import org.springframework.web.bind.annotation.RequestParam;
   
   @Controller
   @RequestMapping("/shiro")
   public class ShiroHandler {
   
       @RequestMapping("/login")
       public String login(@RequestParam("username")String username,@RequestParam("password") String password){
           Subject currentUser = SecurityUtils.getSubject();
           if (!currentUser.isAuthenticated()) {
               // 把用户名和密码封装为UsernamePasswordToken对象
               UsernamePasswordToken token = new UsernamePasswordToken(username, password);
               // 记住我设置
               token.setRememberMe(true);
               try {
                   // 执行登录
                   currentUser.login(token);
               } catch (AuthenticationException ae) {
                   System.out.println("登录失败："+ae.getMessage());
               }
           }
           return "redirect:/list.jsp";
       }
   
   }
   ```

3. 创建一个自定义的Realm继承AuthenticatingRealm类：

   ```java
   package top.tomxwd.shiro.realms;
   
   import org.apache.shiro.authc.AuthenticationException;
   import org.apache.shiro.authc.AuthenticationInfo;
   import org.apache.shiro.authc.AuthenticationToken;
   import org.apache.shiro.realm.AuthenticatingRealm;
   
   public class ShiroRealm extends AuthenticatingRealm {
       @Override
       protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
           System.out.println("ShiroRealm.doGetAuthenticationInfo:"+authenticationToken);
           return null;
       }
   }
   ```

4. 需要把登录的url打开，否则会被拦截请求，一直跳转到login.jsp页面，且doGetAuthenticationInfo不生效；

   spring-shiro.xml

   ```xml
   <property name="filterChainDefinitions">
       <value>
           /login.jsp = anon
           /shiro/login = anon
           # everything else requires authentication:
           /** = authc
       </value>
   </property>
   ```

5. 运行
