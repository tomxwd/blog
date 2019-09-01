---
title: 17_Shiro_授权流程分析
date: 2019-08-30 11:14:44
tags: 
 - Java
 - Shiro
categories:
 - Shiro
---

# 17_Shiro_授权流程分析

1. 授权需要继承AuthorizingRealm类并实现其doGetAuthorizationInfo()方法
2. AuthorizingRealm类其实是继承自AuthenticatingRealm类的，但没有实现AuthenticatingRealm类中的doGetAuthenticationInfo()方法，所以认证和授权只需要继承AuthorizingRealm类即可；同时实现两个抽象方法即可；

以下代码仅仅是为了演示继承后需要实现的方法：

```java
public class TestRealm extends AuthorizingRealm {

    /**
     * 用于授权的方法
     * @param principalCollection
     * @return
     */
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        return null;
    }

    /**
     * 用于认证的方法
     * @param authenticationToken
     * @return
     * @throws AuthenticationException
     */
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
        return null;
    }
}
```

