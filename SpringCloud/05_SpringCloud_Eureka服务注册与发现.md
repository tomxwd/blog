---
title: 05_SpringCloud_Eureka服务注册与发现
date: 2019-12-06 22:32:47
tags: 
 - SpringCloud
categories:
 - SpringCloud
---

# 05_SpringCloud_Eureka服务注册与发现

## Eureka是什么

- Eureka是Netflix的一个子模块，也是核心模块之一。Eureka是一个基于REST的服务，用于定位服务，以实现云端中间层服务发现和故障转移。服务注册与发现对于微服务架构来说是非常重要的，有了服务发现与注册，**只需要使用服务的标识符，就可以访问到服务**，而不需要修改服务调用的配置文件了。**功能类似于dubbo的注册中心，比如zookeeper**；
- Netfilx在设计Eureka时遵守的就是AP原则



## Eureka原理讲解

- Eureka的基本架构

  - SpringCloud封装了Netflix公司开发的Eureka模块来**实现服务注册与发现**；
  - Eureka采用了C/S的设计架构。Eureka Server作为服务注册功能的服务器，他是服务注册中心；
  - 而系统中的其他微服务，使用Eureka的客户端连接到Eureka Server并维持心跳连接。这样系统的维护人员就可以通过Eureka Server来监控系统中各个微服务是否正常运行。SpringCloud的一些其他模块（比如Zuul）就可以通过Eureka Server来发现系统中的其他微服务，并执行相关的逻辑；

  ![Eureka架构](05_SpringCloud_Eureka%E6%9C%8D%E5%8A%A1%E6%B3%A8%E5%86%8C%E4%B8%8E%E5%8F%91%E7%8E%B0/image-20191206224222760.png)

  ![Dubbo架构](05_SpringCloud_Eureka%E6%9C%8D%E5%8A%A1%E6%B3%A8%E5%86%8C%E4%B8%8E%E5%8F%91%E7%8E%B0/image-20191206224247208.png)

  - **Eureka包含两个组件：Eureka Server 和 Eureka Client**

    - Eureka Server提供服务注册服务

      各个节点启动后，会在Eureka Server中进行注册，这样Eureka Server中的服务注册表中将会存储所有可用服务节点的信息，服务节点的信息可以在界面中直观的看到；

    - Eureka Client是一个Java客户端，用于简化Eureka Server的交互

      客户端同时也需要具备一个内置的，使用轮询（round-robin）负载均衡算法的负载均衡器。在应用启动后，将会向Eureka Server发送心跳（默认周期30秒）。如果Eureka Server在多个心跳周期内没有接收到某个节点的心跳，Eureka Server将会从服务注册表中把这个服务节点移除（默认90秒）；

- 三大角色

  - Eureka Server提供服务注册与发现
  - Service Provider服务提供方将自身服务注册到Eureka，从而使服务消费方能够找到
  - Service Consumer服务消费方从Eureka获取注册服务列表，从而能够消费服务



## Eureka 构建步骤

### microservicecloud-eureka-7001

Eureka服务注册中心Module建立

**如果要引入一个cloud的一个新技术组件，基本上有两步：**

- **新增一个相关的maven坐标**

- **再在主启动类上，启动该新组件技术的相关注解标签**



1. 新建microservicecloud-eureka-7001项目

2. pom.xml文件

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
   
       <artifactId>microservicecloud-eureka-7001</artifactId>
   
       <dependencies>
           <!-- eureka-server服务端 -->
           <dependency>
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-starter-eureka-server</artifactId>
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

3. yaml配置文件：

   ```yaml
   server:
     port: 7001
   eureka:
     instance:
       hostname: localhost                        # eureka服务端的实例名称
     client:
       register-with-eureka: false                # false表示不向注册中心注册自己
       fetch-registry: false                      # false表示自己端就是注册中心，职责就是维护服务实例，并不需要去检索服务
       service-url:
         defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/    # 设置与Eureka Server交互的地址查询服务和注册服务都需要依赖这个地址
   ```

4. EurekaServer7001_App主启动类

   ```java
   @SpringBootApplication
   // EurekaServer服务端启动类，接受其他微服务注册进来
   @EnableEurekaServer
   public class EurekaServer7001_App {
   
       public static void main(String[] args) {
           SpringApplication.run(EurekaServer7001_App.class,args);
       }
   
   }
   ```

5. 测试

   - 访问localhost:7001
   -  No instances available 因为没有服务注册进来，所以是空的



### microservicecloud-provider-dept-8001

将已有的部门微服务注册到Eureka服务中心

1. 修改microservicecloud-provider-dept-8001

2. pom.xml

   - 修改部分

     ```xml
     <!-- 将微服务provider注册进Eureka -->
     <dependency>
         <groupId>org.springframework.cloud</groupId>
         <artifactId>spring-cloud-starter-eureka</artifactId>
     </dependency>
     <dependency>
         <groupId>org.springframework.cloud</groupId>
         <artifactId>spring-cloud-starter-config</artifactId>
     </dependency>
     ```

   - 完整内容

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
     
         <artifactId>microservicecloud-provider-dept-8001</artifactId>
     
         <dependencies>
             <dependency>
                 <groupId>top.tomxwd</groupId>
                 <artifactId>microservicecloud-api</artifactId>
                 <version>${project.version}</version>
             </dependency>
             <dependency>
                 <groupId>junit</groupId>
                 <artifactId>junit</artifactId>
             </dependency>
             <dependency>
                 <groupId>mysql</groupId>
                 <artifactId>mysql-connector-java</artifactId>
             </dependency>
             <dependency>
                 <groupId>com.alibaba</groupId>
                 <artifactId>druid</artifactId>
             </dependency>
             <dependency>
                 <groupId>ch.qos.logback</groupId>
                 <artifactId>logback-core</artifactId>
             </dependency>
             <dependency>
                 <groupId>org.mybatis.spring.boot</groupId>
                 <artifactId>mybatis-spring-boot-starter</artifactId>
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
             <!-- 将微服务provider注册进Eureka -->
             <dependency>
                 <groupId>org.springframework.cloud</groupId>
                 <artifactId>spring-cloud-starter-eureka</artifactId>
             </dependency>
             <dependency>
                 <groupId>org.springframework.cloud</groupId>
                 <artifactId>spring-cloud-starter-config</artifactId>
             </dependency>
         </dependencies>
     </project>
     ```

3. application.yaml

   - 修改部分

     ```yaml
     eureka:
       client:                                                   # 将客户端注册进Eureka
         service-url:
           defaultZone: http://localhost:7001/eureka
     ```

   - 完整内容

     ```yaml
     server:
       port: 8001
     
     mybatis:
       config-location: classpath:mybatis/mybatis.cfg.xml        # mybatis配置文件所在路径
       type-aliases-package: top.tomxwd.springcloud.entity       # 所有Entity别名类所在包
       mapper-locations:
         - classpath:mybatis/mapper/**/*.xml                     # mapper映射文件
     spring:
       application:
         name: microservicecloud-dept                            # 应用名
       datasource:
         type: com.alibaba.druid.pool.DruidDataSource            # 当前数据源操作类型
         driver-class-name: org.gjt.mm.mysql.Driver              # mysql驱动包
         url: jdbc:mysql://localhost:3306/cloudDB01?useUnicode=true&characterEncoding=utf8        # 数据库名称
         username: root
         password: root
         dbcp2:
           min-idle: 5                                           # 数据库连接池的最小维持连接数
           initial-size: 5                                       # 初始化连接数
           max-total: 5                                          # 最大连接数
           max-wait-millis: 200                                  # 等待连接获取的最大超时时间
     logging:
       level: debug
     
     eureka:
       client:                                                   # 将客户端注册进Eureka
         service-url:
           defaultZone: http://localhost:7001/eureka
     ```

4. DeptProvider8001_App主启动类

   - 修改部分：

     ```java
     // 本服务启动后会自动注册进Eureka服务中
     @EnableEurekaClient
     ```

   - 完整内容：

     ```java
     @SpringBootApplication
     // 本服务启动后会自动注册进Eureka服务中
     @EnableEurekaClient
     public class DeptProvider8001_App {
     
         public static void main(String[] args) {
             SpringApplication.run(DeptProvider8001_App.class,args);
         }
     
     }
     ```

5. 测试

   - 启动注册中心7001项目以及provider8001项目
   - 访问localhost:7001看是否有应用注册到注册中心



### actuator与注册微服务信息完善



### Eureka自我保护



### microservicecloud-provider-dept-8001

服务发现Discovery















