---
title: 11_Shiro_密码的MD5加密
date: 2019-08-30 08:48:29
tags: 
 - Java
 - Shiro
categories:
 - Shiro
---

# 11_Shiro_密码的MD5加密

1. 如何把一个字符串加密为MD5

   MD5加密123456，迭代2次，无盐值：

   ```java
   public static void main(String[] args) {
       String hashAlgorithmName = "MD5";
       Object credentials = "123456";
       Object salt = null;
       int hashIterations = 2;
       SimpleHash simpleHash = new SimpleHash(hashAlgorithmName, credentials, salt, hashIterations);
       System.out.println("simpleHash = " + simpleHash);
   }
   ```

   结果：

   ```
   simpleHash = 4280d89a5a03f812751f504cc10ee8a5
   ```

   把结果替换到验证流程的，假装数据库获取的密码；

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
           Object credentials="4280d89a5a03f812751f504cc10ee8a5";
           // 3) realmName：当前realm对象的name，调用父类的getName方法即可
           String realmName = getName();
           SimpleAuthenticationInfo simpleAuthenticationInfo = new SimpleAuthenticationInfo(principal, credentials, realmName);
           return simpleAuthenticationInfo;
       }
   }
   ```

2. 替换当前Realm的credentialsMatcher属性，直接使用HashedCredentialsMatcher对象，并设置属性加密算法即可；

   ```xml
   <!--
       3.配置Realm
       3.1 直接配置实现了org.apache.shiro.realm.Realm接口的bean
   -->
   <bean id="jdbcRealm" class="top.tomxwd.shiro.realms.ShiroRealm">
       <property name="credentialsMatcher">
           <bean class="org.apache.shiro.authc.credential.HashedCredentialsMatcher">
               <!-- 指定算法MD5 -->
               <property name="hashAlgorithmName" value="MD5"></property>
               <!-- 迭代次数 -->
               <property name="hashIterations" value="2"></property>
           </bean>
       </property>
   </bean>
   ```


