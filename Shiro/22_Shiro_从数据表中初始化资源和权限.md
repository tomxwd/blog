---
title: 22_Shiro_从数据表中初始化资源和权限
date: 2019-08-30 15:18:24
tags: 
 - Java
 - Shiro
categories:
 - Shiro
---

# 22_Shiro_从数据表中初始化资源和权限

由于用filterChainDefinitions来做URL权限控制在量大的时候会十分不方便，因此我们考虑将权限控制的条目放到数据库中，需要用到权限认证的时候将其取出；



修改spring-shiro.xml配置：

去掉之前的ShiroFilterFactoryBean中的filterChainDefinitions属性，而是注入一个filterChainDefinitionMap属性：

```xml
<!--
   6. 配置ShiroFilter
   6.1 id必须和web.xml文件中配置的org.springframework.web.filter.DelegatingFilterProxy的<filter-name>一致
  -->
<bean id="shiroFilter" class="org.apache.shiro.spring.web.ShiroFilterFactoryBean">
    <property name="securityManager" ref="securityManager"/>
    <property name="loginUrl" value="/login.jsp"/>
    <property name="successUrl" value="/list.jsp"/>
    <property name="unauthorizedUrl" value="/unauthorized.jsp"/>
    <!--
            配置哪些页面需要受保护，
            以及访问这些页面需要的权限
            1） anon：可以被匿名访问
            2） authc：必须认证（即登录）后才可以访问的页面
            3） logout: 登出
            4） roles：角色过滤器
        -->
    <!--<property name="filterChainDefinitions">
            <value>
                /login.jsp = anon
                /shiro/login = anon
                /shiro/logout = logout
                /user.jsp = roles[user]
                /admin.jsp = roles[admin]
                # everything else requires authentication:
                /** = authc
            </value>
        </property>-->
    <property name="filterChainDefinitionMap" ref="filterChainDefinitionMap"></property>
</bean>
```

配置一个bean，该bean实际上是一个Map，通过实例工厂来创建：

```java
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
        map.put("/user.jsp","roles[user]");
        map.put("/admin.jsp","roles[admin]");
        map.put("/**","authc");
        return map;
    }

}
```

配置实例工厂类：

```xml
<!--配置一个bean，该bean实际上是一个Map，通过实例工厂来创建-->
<bean id="filterChainDefinitionMap" factory-bean="filterChainDefinitionMapBuilder"
      factory-method="buildFilterChainDefinitionMap"></bean>
```



