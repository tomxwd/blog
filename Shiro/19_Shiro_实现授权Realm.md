---
title: 19_Shiro_实现授权Realm.md
date: 2019-08-30 13:46:29
tags: 
 - Java
 - Shiro
categories:
 - Shiro
---

# 19_Shiro_实现授权Realm.md

从授权的源码看到doGetAuthorizationInfo方法，多个Realm是按照顺序加进一个LinkedHashSet，保证其有序性，因此我们直接调用getPrimaryPrincipal()方法可以取得下一个Principal信息；

修改ShiroRealm类，继承AuthorizingRealm类并实现doGetAuthorizationInfo()方法：

```java
/**
 * 授权会被shiro回调的方法
 * @param principalCollection
 * @return
 */
@Override
protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
    System.out.println("ShiroRealm.doGetAuthorizationInfo");
    // 1. 从PrincipalCollection中获取登录用户信息
    Object principal = principalCollection.getPrimaryPrincipal();

    // 2. 利用登录用户的信息来获取当前用户的角色或权限（可能需要查询数据库）
    Set<String> roles = new HashSet<>();
    roles.add("user");
    if("admin".equals(principal)){
        roles.add("admin");
    }

    // 3. 创建SimpleAuthorizationInfo，并设置其roles属性
    SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();
    info.setRoles(roles);

    // 4. 返回SimpleAuthorizationInfo对象
    principal = principalCollection.getPrimaryPrincipal();
    return info;
}
```

