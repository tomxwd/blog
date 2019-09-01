---
title: 15_Shiro_把realms配置给SecurityManager
date: 2019-08-30 10:44:11
tags: 
 - Java
 - Shiro
categories:
 - Shiro
---

# 15_Shiro_把realms配置给SecurityManager

这里把realms配置给SecurityManager以便进行下一步授权操作：

```xml
<bean id="securityManager" class="org.apache.shiro.web.mgt.DefaultWebSecurityManager">
    <property name="cacheManager" ref="cacheManager"/>
    <!--<property name="realm" ref="jdbcRealm"/>-->
    <!--<property name="authenticator" ref="authenticator"></property>-->
    <property name="realms">
        <list>
            <ref bean="jdbcRealm"></ref>
            <ref bean="secondRealm"></ref>
        </list>
    </property>
</bean>
```

