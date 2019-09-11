---
title: 12_Dubbo_与SpringBoot整合
date: 2019-09-11 21:56:19
tags: 
 - Dubbo
categories:
 - Java
 - Dubbo
---

# 12_Dubbo_与SpringBoot整合

直接创建SpringBoot项目来进行整合;

1. 导入dubbo-starter

   [dubbo_github对dubbo-starter的说明](https://github.com/apache/dubbo-spring-boot-project)

   ```xml
   <!-- Dubbo Spring Boot Starter -->
   <dependency>
       <groupId>org.apache.dubbo</groupId>
       <artifactId>dubbo-spring-boot-starter</artifactId>
       <version>2.7.3</version>
   </dependency>
   ```

2. 导入dubbo其他依赖，如操作zookeeper的依赖等

   ```xml
   <!-- Dubbo Spring Boot Starter -->
   <dependency>
       <groupId>org.apache.dubbo</groupId>
       <artifactId>dubbo-spring-boot-starter</artifactId>
       <version>2.7.3</version>
   </dependency>
   <!-- curator-framework -->
   <dependency>
       <groupId>org.apache.curator</groupId>
       <artifactId>curator-framework</artifactId>
       <version>4.2.0</version>
   </dependency>
   <!-- curator-recipes -->
   <dependency>
       <groupId>org.apache.curator</groupId>
       <artifactId>curator-recipes</artifactId>
       <version>4.2.0</version>
   </dependency>
   ```

3. 进行dubbo的相关配置，之前我们是用xml来配置的，现在我们用的是SpringBoot项目，所以直接在application.properties中进行相关配置。

   ```properties
   # 应用名
   dubbo.application.name=boot-user-service-provider
   # 注册中心配置
   dubbo.registry.protocol=zookeeper
   dubbo.registry.address=tomxwd.top
   dubbo.registry.port=12343
   # 通信协议 dubbo协议
   dubbo.protocol.name=dubbo
   dubbo.protocol.port=20880
   # 连接监控中心
   dubbo.monitor.protocol=registry
   ```

4. 除了在application.properties中对dubbo进行相关配置之外，我们甚至暴露服务也可以用注解的形式来解决。用dubbo的@Service标签即可【点进去看，会发现这个Service标签也是实现了@Component的功能的，因此是会注入到Spring中的，不用再写Spring的组件注解注入了，多此一举】。

5. 还要在启动类上开启@EnableDubbo，否则dubbo相关注解不会生效



## boot-user-service-provider

创建一个普通的SpringBoot项目：

pom.xml：

添加接口依赖：

```xml
<dependency>
    <groupId>top.tomxwd</groupId>
    <artifactId>gmall-interface</artifactId>
    <version>1.0-SNAPSHOT</version>
</dependency>
```

直接把实现类复制过来即可。



## boot-user-order-provider

创建一个SpringBoot-Web项目：



pom.xml：

添加接口依赖：

```xml
<dependency>
    <groupId>top.tomxwd</groupId>
    <artifactId>gmall-interface</artifactId>
    <version>1.0-SNAPSHOT</version>
</dependency>
```



直接把实现类复制过来，然后把返回值修改一下。



编写controller.OrderController：

```java
@Controller
public class OrderController {

    @Autowired
    private OrderService orderService;

    @RequestMapping("/initOrder")
    @ResponseBody
    public List<UserAddress> initOrder(@RequestParam("uid") String userId){
        return orderService.initOrder(userId);
    }

}
```

