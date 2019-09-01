---
title: 03_Shiro_集成 Spring
date: 2019-08-28 15:25:26
tags: 
 - Java
 - Shiro
categories:
 - Shiro
---

# 03_Shiro_集成 Spring

大致步骤：

1. 加入Spring和Shiro的jar包
2. 配置Spring以及SpringMvc
3. 参照：\shiro-root-1.3.2\samples\spring\的配置：web.xml和Spring的配置文件



## 准备初始web+spring环境

pom.xml

```xml
  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.11</version>
      <scope>test</scope>
    </dependency>
    <!-- spring-mvc -->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-webmvc</artifactId>
      <version>5.0.5.RELEASE</version>
    </dependency>
    <!-- Shiro -->
    <dependency>
      <groupId>org.apache.shiro</groupId>
      <artifactId>shiro-all</artifactId>
      <version>1.3.2</version>
    </dependency>
    <dependency>
      <groupId>log4j</groupId>
      <artifactId>log4j</artifactId>
      <version>1.2.17</version>
    </dependency>
    <dependency>
      <groupId>org.slf4j</groupId>
      <artifactId>slf4j-api</artifactId>
      <version>1.7.20</version>
    </dependency>
    <dependency>
      <groupId>org.slf4j</groupId>
      <artifactId>slf4j-log4j12</artifactId>
      <version>1.7.20</version>
    </dependency>
      
    <!-- servlet -->
    <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>javax.servlet-api</artifactId>
      <version>3.1.0</version>
      <scope>provided</scope>
    </dependency>
      
  </dependencies>
```

web.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app>
  <display-name>Archetype Created Web Application</display-name>

  <servlet>
    <servlet-name>dispatcherServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>classpath:applicationContext.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
  </servlet>
  <servlet-mapping>
    <servlet-name>dispatcherServlet</servlet-name>
    <url-pattern>/</url-pattern>
  </servlet-mapping>

  <filter>
    <filter-name>encodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
      <param-name>encoding</param-name>
      <param-value>utf-8</param-value>
    </init-param>
  </filter>
  <filter-mapping>
    <filter-name>encodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>

</web-app>
```

spring-servlet.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <context:component-scan base-package="top.tomxwd"/>

    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/"></property>
        <property name="suffix" value=".jsp"></property>
    </bean>

    <mvc:annotation-driven/>
    <mvc:default-servlet-handler/>

</beans>
```

applicationContext.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    <import resource="spring-servlet.xml"/>


</beans>
```

log4j.properties

```properties
#
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.
#
log4j.rootLogger=INFO, stdout

log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%d %p [%c] - %m %n

# General Apache libraries
log4j.logger.org.apache=WARN

# Spring
log4j.logger.org.springframework=WARN

# Default Shiro logging
log4j.logger.org.apache.shiro=TRACE

# Disable verbose logging
log4j.logger.org.apache.shiro.util.ThreadContext=WARN
log4j.logger.org.apache.shiro.cache.ehcache.EhCache=WARN
```



## 配置shiro相关

可以参照\shiro-root-1.3.2\samples\spring\的配置；



### 首先配置一个ShiroFilter

web.xml追加内容：

```xml
<!-- ShiroFilter配置 -->
<filter>
    <filter-name>shiroFilter</filter-name>
    <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
    <init-param>
        <param-name>targetFilterLifecycle</param-name>
        <param-value>true</param-value>
    </init-param>
</filter>

<filter-mapping>
    <filter-name>shiroFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```



### 再在Spring的配置里面配置Shiro



#### 配置SecurityManager

```xml
<!-- =========================================================
     Shiro Core Components - Not Spring Specific
     ========================================================= -->
<!-- Shiro's main business-tier object for web-enabled applications
     (use DefaultSecurityManager instead when there is no web environment)-->
<!--
    1. 配置SecurityManager
-->
<bean id="securityManager" class="org.apache.shiro.web.mgt.DefaultWebSecurityManager">
    <property name="cacheManager" ref="cacheManager"/>
    <property name="realm" ref="jdbcRealm"/>
</bean>
```



#### 配置CacheManager

```xml
<!-- Let's use some enterprise caching support for better performance.  You can replace this with any enterprise
     caching framework implementation that you like (Terracotta+Ehcache, Coherence, GigaSpaces, etc -->
<!--
    2.配置CacheManager缓存管理器
    2.1 需要加入ehcache的jar包以及配置文件
-->
<bean id="cacheManager" class="org.apache.shiro.cache.ehcache.EhCacheManager">
    <!-- Set a net.sf.ehcache.CacheManager instance here if you already have one.  If not, a new one
         will be creaed with a default config:
         <property name="cacheManager" ref="ehCacheManager"/> -->
    <!-- If you don't have a pre-built net.sf.ehcache.CacheManager instance to inject, but you want
         a specific Ehcache configuration to be used, specify that here.  If you don't, a default
         will be used.: -->
    <property name="cacheManagerConfigFile" value="classpath:ehcache.xml"/>
</bean>
```

- 导入jar：

  ```xml
  <!-- ehcache -->
  <dependency>
      <groupId>net.sf.ehcache</groupId>
      <artifactId>ehcache-core</artifactId>
      <version>2.6.11</version>
  </dependency>
  ```

- 配置文件ehcache.xml：

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:noNamespaceSchemaLocation="http://ehcache.org/ehcache.xsd"
           updateCheck="false">
      <!--
         diskStore：为缓存路径，ehcache分为内存和磁盘两级，此属性定义磁盘的缓存位置。参数解释如下：
         user.home – 用户主目录
         user.dir  – 用户当前工作目录
         java.io.tmpdir – 默认临时文件路径
       -->
      <diskStore path="java.io.tmpdir/Tmp_EhCache"/>
      <!--
         defaultCache：默认缓存策略，当ehcache找不到定义的缓存时，则使用这个缓存策略。只能定义一个。
       -->
      <!--
        name:缓存名称。
        maxElementsInMemory:缓存最大数目
        maxElementsOnDisk：硬盘最大缓存个数。
        eternal:对象是否永久有效，一但设置了，timeout将不起作用。
        overflowToDisk:是否保存到磁盘，当系统当机时
        timeToIdleSeconds:设置对象在失效前的允许闲置时间（单位：秒）。仅当eternal=false对象不是永久有效时使用，可选属性，默认值是0，也就是可闲置时间无穷大。
        timeToLiveSeconds:设置对象在失效前允许存活时间（单位：秒）。最大时间介于创建时间和失效时间之间。仅当eternal=false对象不是永久有效时使用，默认是0.，也就是对象存活时间无穷大。
        diskPersistent：是否缓存虚拟机重启期数据 Whether the disk store persists between restarts of the Virtual Machine. The default value is false.
        diskSpoolBufferSizeMB：这个参数设置DiskStore（磁盘缓存）的缓存区大小。默认是30MB。每个Cache都应该有自己的一个缓冲区。
        diskExpiryThreadIntervalSeconds：磁盘失效线程运行时间间隔，默认是120秒。
        memoryStoreEvictionPolicy：当达到maxElementsInMemory限制时，Ehcache将会根据指定的策略去清理内存。默认策略是LRU（最近最少使用）。你可以设置为FIFO（先进先出）或是LFU（较少使用）。
        clearOnFlush：内存数量最大时是否清除。
        memoryStoreEvictionPolicy:可选策略有：LRU（最近最少使用，默认策略）、FIFO（先进先出）、LFU（最少访问次数）。
        FIFO，first in first out，这个是大家最熟的，先进先出。
        LFU， Less Frequently Used，就是上面例子中使用的策略，直白一点就是讲一直以来最少被使用的。如上面所讲，缓存的元素有一个hit属性，hit值最小的将会被清出缓存。
        LRU，Least Recently Used，最近最少使用的，缓存的元素有一个时间戳，当缓存容量满了，而又需要腾出地方来缓存新的元素的时候，那么现有缓存元素中时间戳离当前时间最远的元素将被清出缓存。
     -->
      <defaultCache
              eternal="false"
              maxElementsInMemory="10000"
              overflowToDisk="false"
              diskPersistent="false"
              timeToIdleSeconds="1800"
              timeToLiveSeconds="259200"
              memoryStoreEvictionPolicy="LRU"/>
  
      <cache
              name="cloud_user"
              eternal="false"
              maxElementsInMemory="5000"
              overflowToDisk="false"
              diskPersistent="false"
              timeToIdleSeconds="1800"
              timeToLiveSeconds="1800"
              memoryStoreEvictionPolicy="LRU"/>
  
  </ehcache>
  ```



#### 配置Realm

```xml
<!-- Used by the SecurityManager to access security data (users, roles, etc).
     Many other realm implementations can be used too (PropertiesRealm,
     LdapRealm, etc. -->
<!--
    3.配置Realm
    3.1 直接配置实现了org.apache.shiro.realm.Realm接口的bean
-->
<bean id="jdbcRealm" class="top.tomxwd.shiro.realms.ShiroRealm">

</bean>
```

需要自己写一个Realm，实现Shiro给的org.apache.shiro.realm.Realm接口：

```java
package top.tomxwd.shiro.realms;

import org.apache.shiro.authc.AuthenticationException;
import org.apache.shiro.authc.AuthenticationInfo;
import org.apache.shiro.authc.AuthenticationToken;
import org.apache.shiro.realm.Realm;

public class ShiroRealm implements Realm {
    @Override
    public String getName() {
        return null;
    }

    @Override
    public boolean supports(AuthenticationToken authenticationToken) {
        return false;
    }

    @Override
    public AuthenticationInfo getAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
        return null;
    }
}
```

#### 配置LifecycleBeanPostProcessor

bean的生命周期后置处理器，可以自动的来调用配置在Spring IOC容器中shiro bean的生命周期方法；

```xml
<!-- =========================================================
     Shiro Spring-specific integration
     ========================================================= -->
<!-- Post processor that automatically invokes init() and destroy() methods
     for Spring-configured Shiro objects so you don't have to
     1) specify an init-method and destroy-method attributes for every bean
        definition and
     2) even know which Shiro objects require these methods to be
        called. -->
<!--
    4. 配置LifecycleBeanPostProcessor，生命周期的bean后置处理器，
    可以自动的来调用配置在Spring IOC容器中shiro bean的生命周期方法
 -->
<bean id="lifecycleBeanPostProcessor" class="org.apache.shiro.spring.LifecycleBeanPostProcessor"/>
```

#### 配置让Spring里的bean可以使用Shiro的注解

仅在配置了LifecycleBeanPostProcessor（bean的生命周期后置处理器）之后，才可以生效；

```xml
<!-- Enable Shiro Annotations for Spring-configured beans.  Only run after
         the lifecycleBeanProcessor has run: -->
<!--
        5. 启用IOC容器中使用Shiro的注解，但必须在配置了LifecycleBeanPostProcessor之后才可以使用
    -->
<bean class="org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator"
      depends-on="lifecycleBeanPostProcessor"/>
<bean class="org.apache.shiro.spring.security.interceptor.AuthorizationAttributeSourceAdvisor">
    <property name="securityManager" ref="securityManager"/>
</bean>
```

#### 配置ShiroFilter

```xml
	<!-- Define the Shiro Filter here (as a FactoryBean) instead of directly in web.xml -
         web.xml uses the DelegatingFilterProxy to access this bean.  This allows us
         to wire things with more control as well utilize nice Spring things such as
         PropertiesPlaceholderConfigurer and abstract beans or anything else we might need: -->
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
     -->
    <property name="filterChainDefinitions">
        <value>
            /login.jsp = anon
            # everything else requires authentication:
            /** = authc
        </value>
    </property>
</bean>
```

1. id必须和web.xml文件中配置的org.springframework.web.filter.DelegatingFilterProxy的`<filter-name>`一致。

2. 登录页面：

   login.jsp：

   ```jsp
   <%@ page contentType="text/html;charset=UTF-8" language="java" %>
   <html>
   <head>
       <title>Login</title>
   </head>
   <body>
       <h4>Login Page</h4>
   </body>
   </html>
   ```

3. 登录成功页面：

   list.jsp：

   ```jsp
   <%@ page contentType="text/html;charset=UTF-8" language="java" %>
   <html>
   <head>
       <title>List</title>
   </head>
   <body>
   <h4>List Page</h4>
   </body>
   </html>
   ```

4. 没有权限页面：

   unauthorized.jsp：

   ```jsp
   <%@ page contentType="text/html;charset=UTF-8" language="java" %>
   <html>
   <head>
       <title>Unauthorized</title>
   </head>
   <body>
   <h4>Unauthorized Page</h4>
   </body>
   </html>
   ```

5. 配置哪些页面需要受保护，以及访问这些页面需要的权限

   1） anon：可以被匿名访问
   2） authc：必须认证（即登录）后才可以访问的页面



## 总配置文件

spring-shiro.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">


    <!-- =========================================================
         Shiro Core Components - Not Spring Specific
         ========================================================= -->
    <!-- Shiro's main business-tier object for web-enabled applications
         (use DefaultSecurityManager instead when there is no web environment)-->
    <!--
        1. 配置SecurityManager
    -->
    <bean id="securityManager" class="org.apache.shiro.web.mgt.DefaultWebSecurityManager">
        <property name="cacheManager" ref="cacheManager"/>
        <property name="realm" ref="jdbcRealm"/>
    </bean>

    <!-- Let's use some enterprise caching support for better performance.  You can replace this with any enterprise
         caching framework implementation that you like (Terracotta+Ehcache, Coherence, GigaSpaces, etc -->
    <!--
        2.配置CacheManager缓存管理器
        2.1 需要加入ehcache的jar包以及配置文件
    -->
    <bean id="cacheManager" class="org.apache.shiro.cache.ehcache.EhCacheManager">
        <!-- Set a net.sf.ehcache.xml.CacheManager instance here if you already have one.  If not, a new one
             will be creaed with a default config:
             <property name="cacheManager" ref="ehCacheManager"/> -->
        <!-- If you don't have a pre-built net.sf.ehcache.xml.CacheManager instance to inject, but you want
             a specific Ehcache configuration to be used, specify that here.  If you don't, a default
             will be used.: -->
        <property name="cacheManagerConfigFile" value="classpath:ehcache.xml"/>
    </bean>

    <!-- Used by the SecurityManager to access security data (users, roles, etc).
         Many other realm implementations can be used too (PropertiesRealm,
         LdapRealm, etc. -->
    <!--
        3.配置Realm
        3.1 直接配置实现了org.apache.shiro.realm.Realm接口的bean
    -->
    <bean id="jdbcRealm" class="top.tomxwd.shiro.realms.ShiroRealm">

    </bean>

    <!-- =========================================================
         Shiro Spring-specific integration
         ========================================================= -->
    <!-- Post processor that automatically invokes init() and destroy() methods
         for Spring-configured Shiro objects so you don't have to
         1) specify an init-method and destroy-method attributes for every bean
            definition and
         2) even know which Shiro objects require these methods to be
            called. -->
    <!--
        4. 配置LifecycleBeanPostProcessor，生命周期的bean后置处理器，
        可以自动的来调用配置在Spring IOC容器中shiro bean的生命周期方法
     -->
    <bean id="lifecycleBeanPostProcessor" class="org.apache.shiro.spring.LifecycleBeanPostProcessor"/>

    <!-- Enable Shiro Annotations for Spring-configured beans.  Only run after
         the lifecycleBeanProcessor has run: -->
    <!--
        5. 启用IOC容器中使用Shiro的注解，但必须在配置了LifecycleBeanPostProcessor之后才可以使用
    -->
    <bean class="org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator"
          depends-on="lifecycleBeanPostProcessor"/>
    <bean class="org.apache.shiro.spring.security.interceptor.AuthorizationAttributeSourceAdvisor">
        <property name="securityManager" ref="securityManager"/>
    </bean>

    <!-- Define the Shiro Filter here (as a FactoryBean) instead of directly in web.xml -
         web.xml uses the DelegatingFilterProxy to access this bean.  This allows us
         to wire things with more control as well utilize nice Spring things such as
         PropertiesPlaceholderConfigurer and abstract beans or anything else we might need: -->
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
        -->
        <property name="filterChainDefinitions">
            <value>
                /login.jsp = anon
                # everything else requires authentication:
                /** = authc
            </value>
        </property>
    </bean>

</beans>
```







