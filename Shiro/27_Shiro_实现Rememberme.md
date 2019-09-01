---
title: 27_Shiro_实现Rememberme.md
date: 2019-08-30 16:19:38
tags: 
 - Java
 - Shiro
categories:
 - Shiro
---

# 27_Shiro_实现Rememberme.md

## 身份认证相关的拦截器

![身份认证相关的拦截器](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/Shiro/27%E8%BA%AB%E4%BB%BD%E8%AE%A4%E8%AF%81%E7%9B%B8%E5%85%B3%E7%9A%84%E6%8B%A6%E6%88%AA%E5%99%A8.png)



## 改进原本写在xml配置中的拦截配置

先改FilterChainDefinitionMapBuilder返回的Map：

主要加map.put("/list.jsp","user")；

```java
package top.tomxwd.shiro.factory;

import org.springframework.stereotype.Component;

import java.util.LinkedHashMap;

@Component
public class FilterChainDefinitionMapBuilder {

    /**
     * <value>
     *     /login.jsp = anon
     *     /shiro/login = anon
     *     /shiro/logout = logout
     *     /user.jsp = roles[user]
     *     /admin.jsp = roles[admin]
     *     # everything else requires authentication:
     *     /** = authc
     * </value>
     * @return
     */
    public LinkedHashMap<String,String> buildFilterChainDefinitionMap(){
        LinkedHashMap<String,String> map = new LinkedHashMap<>();
        map.put("/login.jsp","anon");
        map.put("/shiro/login","anon");
        map.put("/shiro/logout","logout");
        map.put("/user.jsp","authc,roles[user]");
        map.put("/admin.jsp","authc,roles[admin]");
        map.put("/list.jsp","user");
        map.put("/**","authc");
        return map;
    }

}
```

在login方法中加入：

```java
// 记住我设置
token.setRememberMe(true);
```

测试流程：

1. 登录，访问admin Page和user Page以及list Page，然后关闭浏览器。
2. 打开浏览器，再次访问，发现admin Page和user Page需要重新登录，而list Page不用，则成功；



## 需要继续优化：

1. 前台页面应该提供一个记住我的复选框，选上了才进行记住我设置。
2. 设置rememberMeManager，可以设置里面的Cookie存活时间等；