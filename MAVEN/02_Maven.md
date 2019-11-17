---
title: MAVEN实战
date: 2019-06-07 18:00:28
tags: 
 - Java
 - Maven
categories:
 - Java环境
 - Maven
---

# MAVEN实战

> 1.传递依赖冲突解决（了解）
> 2.通过maven整合框架（重点）
> 3.通过maven项目进行拆分、聚合（重点）
> 4.私服应用（了解）
---
## 传递依赖冲突解决
传递依赖：
1. A（项目）依赖B，B依赖C**（1.1版本）**，B是A的直接依赖，C就是A的传递依赖。
2. 导入依赖D，D依赖C**（1.2版本）**。

那么如何解决上述问题？
1. Maven自己调解原则
  - **第一声明者优先原则**
    - 即是使用先定义的，比如上述问题中，如果在pom文件中先声明了依赖B，再声明依赖D，则C的版本使用的是B依赖的C版本，是1.1版本。
  - **路径近者优先原则**
    - 直接依赖>间接依赖，就是在pom文件中直接指定版本。
2. 排除依赖
  - **在管理依赖的界面右键排除**
  - **在pom.xml文件的`<dependency>`标签中加入`<exclusions>`标签来声明排除的依赖**
  ```
  <dependency>
    <groupId>org.apache.struts</groupId>
    <artifactId>struts2-spring-plugin</artifactId>
    <version>2.3.24</version>
    <exclusions>
      <exclusion>
        <artifactId>spring-beans</artifactId>
        <groupId>org.springframeworks</groupId>
      </exclusion>
    </exclusions>
  </dependency>
  ```
3. 版本锁定(**推荐使用**)
```
<dependencyManagement>
  <dependencies>
    <dependency>
      <artifactId>org.springframeworks</artifactId>
      <groupId>spring-beans</groupId>
      <version>4.2.4</version>
    </dependency>
  </dependencies>
</dependencyManagement>
```
需要注意的是`<dependencyManagement>`并不会导入依赖，只是管理依赖版本，锁定版本。
4. 版本信息管理，用`<properties>`标签
```
<!-- 属性 -->
<properties>
  <a-version>1.2.2</a-version>
  <spring-version>4.2.4.RELEASE</spring-version>
<properties>
```
在需要版本号的时候用${a-version}来引用(OGNL表达式)。如果下次要改版本，则直接改属性值即可。
---
## ssh配置文件加载顺序
1. 首先部署到tomcat，那么tomcat启用的时候需要什么配置文件？（**web.xml**）。
2. web.xml中会有监听器的信息
```
<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
```
用于监听servletContext对象，web项目中最大的一个对象，这个对象创建了之后，认为项目启动了，项目启动了就要去加载spring的配置文件，而spring的配置是要让对象底层用反射来构建的，所以耗时大，因此要让他在项目启动的时候就去完成对象的反射构建**applicationContext.xml**。
  - 加载属性文件.properties文件
  - 配置DataSource数据源
  - 配置sessionFactory对象：LocalSessionFactoryBean
    - 内部注入数据源：注入Hibernate核心配置文件:hibernate.cfg.xml
      - 引用映射文件
  - 配置事务管理：xml/注解方式
  - dao对象
  - service对象
  - action对象（多例）多例的不会创建，在使用的时候才会创建
3. web.xml中的过滤器信息
```
<fiter-class>org.apache.struts2.dispatcher.ng.filter.StrutsPrepareAndExecuteFilter</fiter-class>
```
用于处理用户的请求，加载struts.xml配置文件。
---
## 通过maven整合SSH框架（重点）
  1. 搭建struts2环境

    - 创建strut2配置文件：struts.xml
    - 在web.xml中配置struts2的核心过滤器
  2. 搭建spring环境

    - 创建spring配置文件applicationContext.xml
    - 在web.xml中配置监听器：ContextLoaderListener
    ```
    <!-- 配置监听器：默认加载WEB-INF/applicationContext.xml文件 -->
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    <!-- 通过上下文参数指定spring配置文件路径 -->
    <context-param>
      <param-name>contextConfigLocation<param-name>
      <param-value>classpath:applicationContext.xml<param-value>
    </context-param>
    ```
  3. 搭建hibernate环境

    - hibernate.cfg.xml
  4. Struts2跟Spring整合

    - 整合关键点：action对象的创建，交给spring创建
      1. 创建action类
      2. 将action对象配置到spring配置文件中
      3. 在struts.xml中，在action节点中class属性配置为spring工厂中action对象的bean id。
  5. Spring跟Hibernate框架整合

    - 整合关键点：
      1. 数据源DataSource交给Spring管理。
      2. SessionFactory对象的创建交给Spring创建。
      3. 事务管理
        - 配置事务管理器：PlatFormTransactionManager接口。
            1. jdbc：DataSourceTransactionManager
            2. Hibernate:HibernateTransactionManager
        - xml方式管理事务
            1. 配置通知：具体增强逻辑
            ```
            <tx:advice id="txAdvice">
              <tx:attributes>
                <!-- 匹配业务类中方法名称 -->
                <tx:method name="save*" />
                <tx:method name="update*" />
                <tx:method name="delete*" />
                <tx:method name="find*" read-only="true"/>
                <tx:method name="*" />
              </tx:attributes>
            </tx:advice>
            ```
            2. 配置aop
            ```
            <aop:config>
              <!-- 配置切点:具体哪些方法需要增强 -->
              <aop:pointcut expression="execution(* top.tomxwd.service.*.*(..))" id="cut"/>
              <!-- 配置切面：将增强逻辑作用到切点上 （通知+切入点） -->
              <aop:advisor advice-ref="txAdvice" pointcut-ref="cut" />
            </aop:config>
            ```
          - 注解方式管理事务
          ```
          <!-- 1、开启注解驱动 -->
          <tx:annotation-driven transaction-manager="transationManager" />
          <!-- 2、在service类上或者方法上使用注解@Transactional-->
          ```
  6. 需求

    - 在地址栏中输入action请求，action-service-dao。完成客户查询。
  7. 具体实现

    - 创建客户实体类、映射文件、将映射文件引入Hibernate核心配置文件中。
    - 创建action、service、dao。完成注入。
      - 在类中添加属性生成set方法。
      - 在spring配置文件中完成注入。（也可以用注解形式）
    - 在struts.xml中配置action。
    -
---
## 总结
1. 页面提交参数，在服务端接收参数。
2. 调用业务层方法->dao的方法->DB。
3. 将返回的数据存值栈。
4. 配置结果视图，挑转页面。
