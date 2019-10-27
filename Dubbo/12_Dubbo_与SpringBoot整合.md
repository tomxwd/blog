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

   这里发现dubbo的starter是有依赖冲突的，所以排除掉，自己引入一个zookeeper，不排除其实并不影响使用。

   ```xml
   <!-- Dubbo Spring Boot Starter -->
   <dependency>
       <groupId>com.alibaba.boot</groupId>
       <artifactId>dubbo-spring-boot-starter</artifactId>
       <version>0.2.0</version>
       <exclusions>
           <exclusion>
               <artifactId>zookeeper</artifactId>
               <groupId>org.apache.zookeeper</groupId>
           </exclusion>
       </exclusions>
   </dependency>
   
   <dependency>
       <groupId>org.apache.zookeeper</groupId>
       <artifactId>zookeeper</artifactId>
       <version>3.4.9</version>
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

4. 除了在application.properties中对dubbo进行相关配置之外，我们甚至暴露服务也可以用注解的形式来解决。用dubbo的@Service标签即可；

5. 还要在启动类上开启@EnableDubbo，否则dubbo相关注解不会生效



## boot-user-service-provider

创建一个普通的SpringBoot项目：



pom.xml：

添加接口依赖：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>

    <dependency>
        <groupId>top.tomxwd</groupId>
        <artifactId>gmall-interface</artifactId>
        <version>1.0-SNAPSHOT</version>
    </dependency>

    <!-- Dubbo Spring Boot Starter -->
    <dependency>
        <groupId>com.alibaba.boot</groupId>
        <artifactId>dubbo-spring-boot-starter</artifactId>
        <version>0.2.0</version>
        <exclusions>
            <exclusion>
                <artifactId>zookeeper</artifactId>
                <groupId>org.apache.zookeeper</groupId>
            </exclusion>
        </exclusions>
    </dependency>

    <dependency>
        <groupId>org.apache.zookeeper</groupId>
        <artifactId>zookeeper</artifactId>
        <version>3.4.9</version>
    </dependency>

</dependencies>
```



application.properties：

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



BootUserServiceProviderApplication：

```java
@SpringBootApplication
@EnableDubbo
public class BootUserServiceProviderApplication {

    public static void main(String[] args) {
        SpringApplication.run(BootUserServiceProviderApplication.class, args);
    }

}
```



UserServiceImpl：

```java
@Service
@org.springframework.stereotype.Service
public class UserServiceImpl implements UserService {

    @Override
    public List<UserAddress> getUserAddressList(String userId) {
        System.out.println("UserServiceImpl.....old...");
        // TODO Auto-generated method stub
        UserAddress address1 = new UserAddress(1, "北京市昌平区宏福科技园综合楼3层", "1", "李老师", "010-56253825", "Y");
        UserAddress address2 = new UserAddress(2, "深圳市宝安区西部硅谷大厦B座3层（深圳分校）", "1", "王老师", "010-56253825", "N");
        return Arrays.asList(address1,address2);
    }

}
```





## boot-user-order-provider

创建一个SpringBoot-Web项目：



pom.xml：

添加接口依赖：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>

    <dependency>
        <groupId>top.tomxwd</groupId>
        <artifactId>gmall-interface</artifactId>
        <version>1.0-SNAPSHOT</version>
    </dependency>

    <!-- Dubbo Spring Boot Starter -->
    <dependency>
        <groupId>com.alibaba.boot</groupId>
        <artifactId>dubbo-spring-boot-starter</artifactId>
        <version>0.2.0</version>
        <exclusions>
            <exclusion>
                <artifactId>zookeeper</artifactId>
                <groupId>org.apache.zookeeper</groupId>
            </exclusion>
        </exclusions>
    </dependency>
    <dependency>
        <groupId>org.apache.zookeeper</groupId>
        <artifactId>zookeeper</artifactId>
        <version>3.4.9</version>
    </dependency>

</dependencies>
```



application.properties：

```properties
# 应用名
dubbo.application.name=boot-order-service-consumer
# 注册中心配置
dubbo.registry.protocol=zookeeper
dubbo.registry.address=tomxwd.top
dubbo.registry.port=12343
# 连接监控中心
dubbo.monitor.protocol=registry

server.port=8081
```



BootOrderServiceConsumerApplication：

```java
@SpringBootApplication
@EnableDubbo
public class BootOrderServiceConsumerApplication {

    public static void main(String[] args) {
        SpringApplication.run(BootOrderServiceConsumerApplication.class, args);
    }

}
```



OrderController：

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



OrderServiceImpl：

```java
@Service
public class OrderServiceImpl implements OrderService {

    @Reference
    UserService userService;

    @Override
    public List<UserAddress> initOrder(String userId) {
        // 1、查询用户的收货地址
        List<UserAddress> addressList = userService.getUserAddressList(userId);
        System.out.println("addressList = " + addressList);
        return addressList;
    }

}
```

