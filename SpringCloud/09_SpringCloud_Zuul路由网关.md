---
title: 09_SpringCloud_Zuul路由网关
date: 2019-12-08 19:18:36
tags: 
 - SpringCloud
categories:
 - SpringCloud
---

# 09_SpringCloud_Zuul路由网关

## Zuul概述

[github资料](https://github.com/Netflix/zuul/wiki)

 ![img](09_SpringCloud_Zuul%E8%B7%AF%E7%94%B1%E7%BD%91%E5%85%B3/diagram-distributed-systems.svg) 

可以从图上看到，Zuul主要就是API Gateway那块，言下之意就是所有的访问必须要经过API Gateway才能访问到微服务；

- Zuul包含了对请求的路由和过滤两个最主要的功能：

- 其中路由功能负责对外部请求转发到具体的微服务实例上，是实现外部访问统一入口的基础；
- 而过滤器功能则负责对请求的处理过程进行干预，是实现请求校验、服务聚合等功能的基础；

- Zuul和Eureka进行整合，将Zuul自身注册为Eureka服务治理下的应用，同时从Eureka中获得其他微服务的信息，也即以后的访问微服务都是通过Zuul跳转后获得的；



**注意：Zuul服务最终还是会注册进Eureka；**



**Zuul提供=代理+路由+过滤三大功能；**



## Zuul路由基本配置

- 新建microservicecloud-zuul-gateway-9527项目

- pom.xml

  - 修改部分：

    ```xml
    <!-- Zuul路由网关相关依赖 -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-eureka</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-zuul</artifactId>
    </dependency>
    ```

  - 完整内容：

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <project xmlns="http://maven.apache.org/POM/4.0.0"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
        <parent>
            <artifactId>microservicecloud</artifactId>
            <groupId>top.tomxwd</groupId>
            <version>1.0-SNAPSHOT</version>
        </parent>
        <modelVersion>4.0.0</modelVersion>
    
        <artifactId>microservicecloud-zuul-gateway-9527</artifactId>
    
        <dependencies>
            <dependency>
                <groupId>top.tomxwd</groupId>
                <artifactId>microservicecloud-api</artifactId>
                <version>${project.version}</version>
            </dependency>
            <!-- Zuul路由网关相关依赖 -->
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-starter-eureka</artifactId>
            </dependency>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-starter-zuul</artifactId>
            </dependency>
            <!-- actuator监控以及信息配置 -->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-actuator</artifactId>
            </dependency>
            <!-- Hystrix断路器相关依赖 -->
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-starter-hystrix</artifactId>
            </dependency>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-starter-config</artifactId>
            </dependency>
            <!-- 内嵌jetty容器 -->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-jetty</artifactId>
            </dependency>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-web</artifactId>
            </dependency>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-test</artifactId>
            </dependency>
            <!-- 热部署 -->
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>springloaded</artifactId>
            </dependency>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-devtools</artifactId>
            </dependency>
        </dependencies>
    
    </project>
    ```

- application.yaml

  ```yaml
  server:
    port: 9527
  
  spring:
    application:
      name: microservicecloud-zuul-getway
  
  eureka:
    client:
      service-url:
        defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/
    instance:
      instance-id: getaway-9527.com
      prefer-ip-address: true
  
  info:
    app.name: tomxwd-microservicecloud
    company.name: www.tomxwd.top
    build.artifactId: $project.artifactId$
    bulid.version: $project.version$
  ```

- hosts修改

  - 127.0.0.1 myzuul.com

- 主启动类Zuul_9527_StartSpringCloudApp，主要是加上@EnableZuulProxy注解

  ```java
  @SpringBootApplication
  @EnableZuulProxy
  public class Zuul_9527_StartSpringCloudApp {
  
      public static void main(String[] args) {
          SpringApplication.run(Zuul_9527_StartSpringCloudApp.class, args);
      }
  
  }
  ```

- 启动

  - Eureka集群启动
  - microservicecloud-provider-dept-8001
  - 一个路由Zuul
    - 可以到Eureka网址查看注册信息

- 测试

  - 不用路由访问：http://localhost:8001/dept/get/2
  - 启用路由访问：http://myzuul.com:9527/microservicecloud-dept/dept/get/2



## Zuul路由访问映射规则

- 修改工程microservicecloud-zuul-gateway-9527

- 代理名称

  - application.yaml

    - 修改部分：

      ```yaml
      zuul:
        routes:
          mydept.serviceId: microservicecloud-dept
          mydept.path: /mydept/**
      ```

    - 完整内容：

      ```yaml
      server:
        port: 9527
      
      spring:
        application:
          name: microservicecloud-zuul-getway
      
      eureka:
        client:
          service-url:
            defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/
        instance:
          instance-id: getaway-9527.com
          prefer-ip-address: true
      
      
      info:
        app.name: tomxwd-microservicecloud
        company.name: www.tomxwd.top
        build.artifactId: $project.artifactId$
        bulid.version: $project.version$
      zuul:
        routes:
          mydept.serviceId: microservicecloud-dept
          mydept.path: /mydept/**
      ```

      修改之前我们访问的是http://myzuul.com:9527/microservicecloud-dept/dept/get/2，修改之后我们访问http://myzuul.com:9527/mydept/dept/get/2即可；

  - 此时存在的问题

    - 用原来真实的路径也可以访问得到服务

- 原真实服务名忽略

  - application.yaml

    - 修改部分：

      ```yaml
      zuul:
      	ignored-services: microservicecloud-dept
      ```

  - 如果是单个，可以直接写应用名来进行ignored-services，多个可以用"*"

    - 修改部分：

      ```yaml
      zuul:
      	ignored-services: "*"
      ```

- 设置统一公共前缀

  所有的服务前面都要加上/tomxwd；

  - application.yaml

    - 修改部分：

      ```yaml
      zuul:
      	prefix: /tomxwd
      ```

  - 这时候要访问 http://myzuul.com:9527/tomxwd/mydept/dept/get/2 才可以；

- 最后application.yaml文件修改

  ```yaml
  server:
    port: 9527
  
  spring:
    application:
      name: microservicecloud-zuul-getway
  
  eureka:
    client:
      service-url:
        defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/
    instance:
      instance-id: getaway-9527.com
      prefer-ip-address: true
  
  
  info:
    app.name: tomxwd-microservicecloud
    company.name: www.tomxwd.top
    build.artifactId: $project.artifactId$
    bulid.version: $project.version$
  zuul:
    routes:
      mydept.serviceId: microservicecloud-dept
      mydept.path: /mydept/**
    ignored-services: "*"
    prefix: /tomxwd
  #  ignored-services: microservicecloud-dept
  ```

  





