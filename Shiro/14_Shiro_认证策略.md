---
title: 14_Shiro_认证策略
date: 2019-08-30 10:30:09
tags: 
 - Java
 - Shiro
categories:
 - Shiro
---

# 14_Shiro_认证策略

## AuthenticationStrategy

- AuthenticationStrategy接口的默认实现：
  - **FirstSuccessfulStrategy：**只要有一个Realm验证成功即可，只返回第一个Realm身份验证成功的认证信息，其他的忽略；
  - **AtLeastOneSuccessfulStrategy：**只要有一个Realm验证成功即可，和FirstSuccessfulStrategy不同，将返回所有Realm身份验证成功的认证信息；
  - **AllSuccessfulStrategy：**所有Realm验证成功才算成功，且返回所有Realm身份验证成功的认证信息，如果有一个失败就失败了。
  - **ModularRealmAuthenticator默认是AtLeastOneSuccessfulStrategy策略；**

默认不配置的话策略是只要有一个Realm验证成功即可；

而修改策略的方式是在定义认证器的时候配置的：

可以直接找到AbstractAuthenticationStrategy抽象类看其子类

```xml
<bean id="authenticator" class="org.apache.shiro.authc.pam.ModularRealmAuthenticator">
    <property name="realms">
        <list>
            <ref bean="jdbcRealm"></ref>
            <ref bean="secondRealm"></ref>
        </list>
    </property>
    <property name="authenticationStrategy">
        <bean class="org.apache.shiro.authc.pam.AllSuccessfulStrategy"></bean>
    </property>
</bean>
```

