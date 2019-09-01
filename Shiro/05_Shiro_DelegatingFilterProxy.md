---
title: 05_Shiro_DelegatingFilterProxy
date: 2019-08-29 16:02:43
tags: 
 - Java
 - Shiro
categories:
 - Shiro
---

# 05_Shiro_DelegatingFilterProxy

org.apache.shiro.spring.web.ShiroFilterFactoryBean的id必须要和web.xml中的DelegatingFilterProxy的`<filter-name>`一致，否则会报错，找不到bean【NoSuchBeanDefinitionException】，因为Shiro会来IOC容器中和filter-name名字对应的filter bean；

其实也可以用targetBeanName初始化参数来指定ioc中的filter bean：

```xml
<!-- ShiroFilter配置 -->
<!--
  1. 配置Shiro的shiroFilter
  2. DelegatingFilterProxy实际上是Filter的一个代理对象，默认情况下，Spring会到IOC容器中查找
  和<filter-name>对应的filter bean，也可以通过targetBeanName的初始化参数来配置filter bean的id
 -->
<filter>
    <filter-name>shiroFilter2</filter-name>
    <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
    <init-param>
        <param-name>targetFilterLifecycle</param-name>
        <param-value>true</param-value>
    </init-param>
    <init-param>
        <param-name>targetBeanName</param-name>
        <param-value>shiroFilter</param-value>
    </init-param>
</filter>

<filter-mapping>
    <filter-name>shiroFilter2</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```





