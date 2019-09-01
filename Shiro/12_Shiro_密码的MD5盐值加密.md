---
title: 12_Shiro_密码的MD5盐值加密
date: 2019-08-30 09:06:28
tags: 
 - Java
 - Shiro
categories:
 - Shiro
---

# 12_Shiro_密码的MD5盐值加密

## 为什么要使用到盐值加密：

为了让同样的密码进行同样的加密之后，产生不一样的md5码，需要引进盐值。



## 如何做到盐值加密：

1. 在doGetAuthenticationInfo()方法返回值创建SimpleAuthenticationInfo对象的时候，需要使用new SimpleAuthenticationInfo(principal, credentials, credentialsSalt, realmName)构造器；

2. 使用ByteSource.Util.bytes()方法来计算盐值；
3. 盐值需要唯一：一般使用随机字符串或user id；
4. 使用new SimpleHash(hashAlgorithmName, credentials, salt, hashIterations)方法来计算盐值加密后的密码；

这里只对admin和user用户进行放行，用到的盐值是其用户名：

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
        Object credentials = null;
        if("admin".equals(username)){
            credentials = "928bfd2577490322a6e19b793691467e";
        }else if("user".equals(username)){
            credentials = "b8c2d5b0a37cc51f91d5e8970347a3a3";
        }
        // 3) realmName：当前realm对象的name，调用父类的getName方法即可
        String realmName = getName();
        // 4) 盐值
        ByteSource credentialsSalt = ByteSource.Util.bytes(username);
        SimpleAuthenticationInfo simpleAuthenticationInfo = new SimpleAuthenticationInfo(principal, credentials, credentialsSalt, realmName);
        return simpleAuthenticationInfo;
    }
}
```



