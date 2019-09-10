---
title: 10_Dubbo_服务消费者配置&测试
date: 2019-09-10 22:22:14
tags: 
 - Dubbo
categories:
 - Java
 - Dubbo
---

# 10_Dubbo_服务消费者配置&测试

让服务消费者去注册中心订阅服务提供者的服务地址（订阅服务）

- 导入dubbo依赖（2.6.2）

  ```xml
  <!-- dubbo -->
  <dependency>
      <groupId>com.alibaba</groupId>
      <artifactId>dubbo</artifactId>
      <version>2.6.2</version>
  </dependency>
  ```

- 引入操作Zookeeper的客户端

  2.6以前引入zkClient：

  ```xml
  <dependency>
      <groupId>com.101tec</groupId>
      <artifactId>zkclient</artifactId>
      <version>0.10</version>
  </dependency>
  ```

  2.6以后引入curator-framework:

  ```xml
  <dependency>
      <groupId>org.apache.curator</groupId>
      <artifactId>curator-framework</artifactId>
      <version>2.12.0</version>
  </dependency>
  ```

- 编写provider的配置文件

  1. 指定当前服务/应用的名字（同样的服务名字相同，不要和别的服务重名）
  2. 指定注册中心的位置
  4. 声明需要调用的远程服务：生成远程服务代理
  
  order-service-consumer【consumer.xml】
  
  ```xml
<?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
         xmlns:context="http://www.springframework.org/schema/context"
         xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://dubbo.apache.org/schema/dubbo http://dubbo.apache.org/schema/dubbo/dubbo.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
  
      <dubbo:application name="order-service-consumer"></dubbo:application>
  
      <dubbo:registry protocol="zookeeper" address="tomxwd.top" port="12343"></dubbo:registry>
  
      <!-- 声明需要调用的远程服务 :生成远程服务代理-->
      <dubbo:reference interface="top.tomxwd.gmall.service.UserService" id="userService"></dubbo:reference>
  
      <!-- 包扫描 -->
      <context:component-scan base-package="top.tomxwd.gmall"></context:component-scan>
  
  
  </beans>
  ```

- 改一下OrderServiceImpl的代码：

  **注意这里用的Service还是Spring的Service注解**

  这里只是把远程的服务，注入到Spring容器之后，注入到OrderServiceImpl的userService中而已，然后利用测试来获取本地的OrderService进行测试，如果能够调用到userService就代表调用远程服务成功。

  ```java
  @Service
  public class OrderServiceImpl implements OrderService {
  
      @Autowired
      UserService userService;
  
      public void initOrder(String userId) {
          // 1、查询用户的收货地址
          List<UserAddress> addressList = userService.getUserAddressList(userId);
          System.out.println("addressList = " + addressList);
      }
  
  }
  ```

- 测试代码：

  ```java
  public class MainApplication {
  
      public static void main(String[] args) throws IOException {
          ClassPathXmlApplicationContext ioc = new ClassPathXmlApplicationContext("classpath:consumer.xml");
          ioc.start();
          OrderService orderService = ioc.getBean(OrderService.class);
          orderService.initOrder("1");
          System.in.read();
      }
  
  }
  ```

  能正常输出即可。