---
title: 13_Shiro_多Realm验证
date: 2019-08-30 10:15:02
tags: 
 - Java
 - Shiro
categories:
 - Shiro
---

# 13_Shiro_多Realm验证

定义第二个Realm，使用SHA1算法加密；

```java
package top.tomxwd.shiro.realms;

import org.apache.shiro.authc.*;
import org.apache.shiro.crypto.hash.SimpleHash;
import org.apache.shiro.realm.AuthenticatingRealm;
import org.apache.shiro.util.ByteSource;

public class SecondRealm extends AuthenticatingRealm {
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {

        System.out.println("[SecondRealm] doGetAuthenticationInfo()");

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
            credentials = "14c33c244faa5ad403e38be03037060e97bc7aea";
        }else if("user".equals(username)){
            credentials = "9b51165daa869d7d7162480119108f3119e00249";
        }
        // 3) realmName：当前realm对象的name，调用父类的getName方法即可
        String realmName = getName();
        // 4) 盐值
        ByteSource credentialsSalt = ByteSource.Util.bytes(username);
        SimpleAuthenticationInfo simpleAuthenticationInfo = new SimpleAuthenticationInfo(principal, credentials, credentialsSalt, realmName);
        return simpleAuthenticationInfo;
    }

    public static void main(String[] args) {
        String hashAlgorithmName = "SHA1";
        Object credentials = "123456";
        Object salt = ByteSource.Util.bytes("user");
        int hashIterations = 2;
        SimpleHash simpleHash = new SimpleHash(hashAlgorithmName, credentials, salt, hashIterations);
        System.out.println("simpleHash = " + simpleHash);
    }
}
```

shiro配置文件中需要声明注入SecondRealm，且需要声明一个**多Realm认证器【ModularRealmAuthenticator】**，且在SecurityManager中奖该认证器配进去：

SecondRealm：

```xml
<bean id="secondRealm" class="top.tomxwd.shiro.realms.SecondRealm">
    <property name="credentialsMatcher">
        <bean class="org.apache.shiro.authc.credential.HashedCredentialsMatcher">
            <property name="hashAlgorithmName" value="SHA1"></property>
            <property name="hashIterations" value="2"></property>
        </bean>
    </property>
</bean>
```

ModularRealmAuthenticator：

```xml
<bean id="authenticator" class="org.apache.shiro.authc.pam.ModularRealmAuthenticator">
    <property name="realms">
        <list>
            <ref bean="jdbcRealm"></ref>
            <ref bean="secondRealm"></ref>
        </list>
    </property>
</bean>
```

SecurityManager：

```xml
<bean id="securityManager" class="org.apache.shiro.web.mgt.DefaultWebSecurityManager">
    <property name="cacheManager" ref="cacheManager"/>
    <property name="authenticator" ref="authenticator"></property>
</bean>
```

当然也可以直接在securityManager中配置realms属性，也是传入list；
