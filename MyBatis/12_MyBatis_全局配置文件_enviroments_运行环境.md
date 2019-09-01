---
title: 12_MyBatis_全局配置文件_enviroments_运行环境
date: 2019-08-09 17:03:28
tags: 
 - Java
 - Mybatis
categories:
 - Mybatis
---

# 12_MyBatis\_全局配置文件\_enviroments_运行环境

> [官网——运行环境](http://www.mybatis.org/mybatis-3/zh/configuration.html#environments)

MyBatis 可以配置成适应多种环境，这种机制有助于将 SQL 映射应用于多种数据库之中， 现实情况下有多种理由需要这么做。例如，开发、测试和生产环境需要有不同的配置；或者想在具有相同 Schema 的多个生产数据库中 使用相同的 SQL 映射。有许多类似的使用场景。

**不过要记住：尽管可以配置多个环境，但每个 SqlSessionFactory 实例只能选择一种环境。**

具体看官网，很详细。



## 细节

每个`<enviroment>`标签里都必须包含两个标签：

- transactionManager：配置事务管理器

  - type：事务管理器的类型;

    在 MyBatis 中有两种类型的事务管理器（也就是 type=”[JDBC|MANAGED]”）

    这两个事务管理器，其实都是别名，具体在Configuration类中可以看到他们具体代表的是哪个事务管理器。

    - JDBC – 这个配置就是直接使用了 JDBC 的提交和回滚设置，它依赖于从数据源得到的连接来管理事务作用域。

    - MANAGED – 这个配置几乎没做什么。它从来不提交或回滚一个连接，而是让容器来管理事务的整个生命周期（比如 JEE 应用服务器的上下文）。 默认情况下它会关闭连接，然而一些容器并不希望这样，因此需要将 closeConnection 属性设置为 false 来阻止它默认的关闭行为。例如:

      ```xml
      <transactionManager type="MANAGED">
        <property name="closeConnection" value="false"/>
      </transactionManager>
      ```

    - 自定义事务管理器：实现TransactionFactory接口，这时候type指定为全类名即可。

- dataSource：数据源

  - type：”[UNPOOLED|POOLED|JNDI]”

    这个也是别名，具体的也可以去Configuration类查看。

    - UNPOOLED：不使用连接池，每次都建立新连接
    - POOLED：连接池技术
    - JNDI：JNDI技术
    - 自定义数据源：实现DataSourceFactory接口。type写全类名。

## 切换环境

`<enviroments>`标签有个default属性，值是`<enviroment>`标签的id属性，而id属性是用于唯一标识，比如开发和测试用的环境不一样，那么就在default中指定不一样的id即可。达到快速切换环境的目的。

```xml
    <environments default="development">
        <environment id="test">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="${jdbc.driver}"/>
                <property name="url" value="${jdbc.url}"/>
                <property name="username" value="${jdbc.username}"/>
                <property name="password" value="${jdbc.password}"/>
            </dataSource>
        </environment>
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="${jdbc.driver}"/>
                <property name="url" value="${jdbc.url}"/>
                <property name="username" value="${jdbc.username}"/>
                <property name="password" value="${jdbc.password}"/>
            </dataSource>
        </environment>
    </environments>
```

