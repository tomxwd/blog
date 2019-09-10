---
title: 09_Dubbo_服务提供者配置&测试
date: 2019-09-09 22:11:06
tags: 
 - Dubbo
categories:
 - Java
 - Dubbo
---

# 09_Dubbo_服务提供者配置&测试

将服务提供者注册到注册中心（暴露服务）

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
  3. 指定通信规则（通信协议？通信端口）protocol【在此端口暴露服务】
     - name：dubbo
     - prot：20880
  4. 声明需要暴露的服务接口
  5. service真正的实现

  user-service-provider【provider.xml】

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
         xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://dubbo.apache.org/schema/dubbo http://dubbo.apache.org/schema/dubbo/dubbo.xsd">
  
      <!-- 1. 指定当前服务/应用的名字（同样的服务名字相同，不要和别的服务重名）-->
      <dubbo:application name="user-service-provider"></dubbo:application>
  
      <!-- 2. 指定注册中心的地址 -->
      <dubbo:registry address="zookeeper://tomxwd.top:12343"></dubbo:registry>
  
      <!-- 3. 指定通信规则（通信协议？通信端口） -->
      <dubbo:protocol name="dubbo" port="20880"></dubbo:protocol>
  
      <!-- 4. 声明需要暴露的服务接口 -->
      <dubbo:service interface="top.tomxwd.gmall.service.UserService" ref="userService"></dubbo:service>
  
      <!-- 5. service真正的实现 -->
      <bean class="top.tomxwd.gmall.service.impl.UserServiceImpl" id="userService"></bean>
  
  </beans>
  ```

- 测试：

  ```java
  public class MainApplication {
  
      public static void main(String[] args) throws IOException {
          ClassPathXmlApplicationContext ioc = new ClassPathXmlApplicationContext("classpath:provider.xml");
          ioc.start();
          System.in.read();
      }
  
  }
  ```

  到dubbo-admin服务治理中心看，可以发现已经注册好了。

