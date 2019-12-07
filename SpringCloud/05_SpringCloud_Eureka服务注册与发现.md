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

**主机名称：服务名称修改**

- 修改microservicecloud-provider-dept-8001

  - application.yaml配置文件

    - 修改部分

      ```yaml
      eureka:
        client:                                                   # 将客户端注册进Eureka
          service-url:
            defaultZone: http://localhost:7001/eureka
        instance:
          instance-id: microservicecloud-dept-8001                # 自定义服务名称信息
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
        instance:
          instance-id: microservicecloud-dept-8001                # 自定义服务名称信息
      ```



**访问信息有IP信息提示**

- 修改microservicecloud-provider-dept-8001

  - application.yaml修改

    - 修改部分：

      ```yaml
      eureka:
        client:                                                   # 将客户端注册进Eureka
          service-url:
            defaultZone: http://localhost:7001/eureka
        instance:
          instance-id: microservicecloud-dept-8001                # 自定义服务名称信息
          prefer-ip-address: true                                 # 访问路径可以显示IP地址
      ```

    - 完整部分：

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
        instance:
          instance-id: microservicecloud-dept-8001                # 自定义服务名称信息
          prefer-ip-address: true                                 # 访问路径可以显示IP地址
      ```

- 修改之后



**微服务info内容详细信息**

在Eureka地址中点击应用发现报404错误，需要解决；

1. 修改microservicecloud-provider-dept-8001的pom.xml文件

   - 修改部分：

     ```xml
     <!-- actuator监控以及信息配置 -->
     <dependency>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter-actuator</artifactId>
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
             <!-- actuator监控以及信息配置 -->
             <dependency>
                 <groupId>org.springframework.boot</groupId>
                 <artifactId>spring-boot-starter-actuator</artifactId>
             </dependency>
         </dependencies>
     </project>
     ```

2. 总的父工程microservicecloud修改pom.xml添加构建build信息

   - 修改部分：

     ```xml
     <!-- 构建项目插件 -->
     <build>
         <finalName>microservicecloud</finalName>
         <resources>
             <resource>
                 <directory>src/main/resources</directory>
                 <filtering>true</filtering>
             </resource>
         </resources>
         <plugins>
             <plugin>
                 <groupId>org.apache.maven.plugins</groupId>
                 <artifactId>maven-resources-plugin</artifactId>
                 <version>3.1.0</version>
                 <!-- 以下信息是指，在扫描上面resource路径下的文件时，遇到$开头$结束的，需要解析，如$project.version$，则会在构建的时候变成项目真实的project.version -->
                 <configuration>
                     <delimiters>
                         <delimit>$</delimit>
                     </delimiters>
                 </configuration>
             </plugin>
         </plugins>
     </build>
     ```

   - 完整内容：

     ```xml
     <?xml version="1.0" encoding="UTF-8"?>
     <project xmlns="http://maven.apache.org/POM/4.0.0"
              xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
              xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
         <modelVersion>4.0.0</modelVersion>
     
         <groupId>top.tomxwd</groupId>
         <artifactId>microservicecloud</artifactId>
         <version>1.0-SNAPSHOT</version>
         <modules>
             <module>microservicecloud-api</module>
             <module>microservicecloud-provider-dept-8001</module>
             <module>microservicecloud-consumer-dept-80</module>
             <module>microservicecloud-eureka-7001</module>
         </modules>
         <packaging>pom</packaging>
     
         <properties>
             <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
             <maven.compiler.source>1.8</maven.compiler.source>
             <maven.compiler.target>1.8</maven.compiler.target>
             <junit.version>4.12</junit.version>
             <log4j.version>1.2.17</log4j.version>
             <lombok.version>1.16.18</lombok.version>
         </properties>
     
         <dependencyManagement>
             <dependencies>
                 <dependency>
                     <groupId>org.springframework.cloud</groupId>
                     <artifactId>spring-cloud-dependencies</artifactId>
                     <version>Dalston.SR1</version>
                     <type>pom</type>
                     <scope>import</scope>
                 </dependency>
                 <dependency>
                     <groupId>org.springframework.boot</groupId>
                     <artifactId>spring-boot-dependencies</artifactId>
                     <version>1.5.9.RELEASE</version>
                     <type>pom</type>
                     <scope>import</scope>
                 </dependency>
                 <dependency>
                     <groupId>mysql</groupId>
                     <artifactId>mysql-connector-java</artifactId>
                     <version>5.0.4</version>
                 </dependency>
                 <dependency>
                     <groupId>com.alibaba</groupId>
                     <artifactId>druid</artifactId>
                     <version>1.0.31</version>
                 </dependency>
                 <dependency>
                     <groupId>org.mybatis.spring.boot</groupId>
                     <artifactId>mybatis-spring-boot-starter</artifactId>
                     <version>1.3.0</version>
                 </dependency>
                 <dependency>
                     <groupId>ch.qos.logback</groupId>
                     <artifactId>logback-core</artifactId>
                     <version>1.2.3</version>
                 </dependency>
                 <dependency>
                     <groupId>junit</groupId>
                     <artifactId>junit</artifactId>
                     <version>${junit.version}</version>
                     <scope>test</scope>
                 </dependency>
                 <dependency>
                     <groupId>log4j</groupId>
                     <artifactId>log4j</artifactId>
                     <version>${log4j.version}</version>
                 </dependency>
             </dependencies>
         </dependencyManagement>
     
         <!-- 构建项目插件 -->
         <build>
             <finalName>microservicecloud</finalName>
             <resources>
                 <resource>
                     <directory>src/main/resources</directory>
                     <filtering>true</filtering>
                 </resource>
             </resources>
             <plugins>
                 <plugin>
                     <groupId>org.apache.maven.plugins</groupId>
                     <artifactId>maven-resources-plugin</artifactId>
                     <version>3.1.0</version>
                     <!-- 以下信息是指，在扫描上面resource路径下的文件时，遇到$开头$结束的，需要解析，如$project.version$，则会在构建的时候变成项目真实的project.version -->
                     <configuration>
                         <delimiters>
                             <delimit>$</delimit>
                         </delimiters>
                     </configuration>
                 </plugin>
             </plugins>
         </build>
     </project>
     ```

3. 修改microservicecloud-provider-dept-8001的application.yaml配置文件

   - 修改部分：

     ```yaml
     info:
       app.name: tomxwd-microservicecloud
       company.name: www.tomxwd.top
       build.artifactId: $project.artifactId$
       bulid.version: $project.version$
     ```

     键值对的形式；

   - 完整内容：

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
       instance:
         instance-id: microservicecloud-dept-8001                # 自定义服务名称信息
         prefer-ip-address: true                                 # 访问路径可以显示IP地址
     info:
       app.name: tomxwd-microservicecloud
       company.name: www.tomxwd.top
       build.artifactId: $project.artifactId$
       bulid.version: $project.version$
     ```



### Eureka自我保护

注册中心的地址过了一段时间会报红，这是Eureka的自我保护；



默认情况下，如果Eureka Server在一定时间内没有接收到某个微服务实例的心跳，Eureka Server将会注销该实例（默认90秒）。

但是当网络分区故障发生的时候，微服务与Eureka Server之间无法正常通信，以上行为可能变得非常危险——因为微服务本身其实是健康的，**此时本不应该注销这个微服务**。Eureka通过“自我保护模式”来解决这个问题——当Eureka Server节点在短时间内丢失过多客户端时（可能发生了网络分区故障），那么这个节点就会进入自我保护模式。一旦进入该模式，Eureka Server就会保护服务注册表中的信息，不再删除服务注册表中的数据（也就是不会注销任何微服务）。当网络故障恢复后，该Eureka Server节点会自动退出自我保护模式。

**在自我保护模式中，Eureka Server会保护服务注册表中的信息，不再注销任何服务实例。当它收到的心跳数重新恢复到阈值以上时，该Eureka Server节点就会自动退出自我保护模式。它的设计哲学就是宁可保留错误的服务注册信息，也不盲目注销任何可能健康的实例。一句话理解就是：好死不如赖活着；**

综上，自我保护模式是一种应对网络异常的安全保护措施。它的架构哲学是宁可同时保留所有微服务（健康的微服务和不健康的微服务都会保留），也不盲目注销任何健康的微服务。使用自我保护模式，可以让Eureka集群更加的健壮、稳定；

在SpringCloud中，可以使用**eureka.server.enable-self-preservation=false**来禁用自我保护模式； 



**演示几个场景**：

1. 修改提供者的配置文件（如服务名称），来回修改，查看注册中心变化；
   - Eureka注册中心检测到服务处于波动，这时候就会报红；
   - 原因：**某时刻某一个微服务不可用了，Eureka不会立刻对其进行清理，依旧会对该微服务的信息进行保存；**
2. 长时间没动静，查看Eureka注册中心变化；



### microservicecloud-provider-dept-8001

服务发现Discovery

- 对于注册进Eureka里面的微服务，可以通过服务发现来获得该服务的信息

- 修改microservicecloud-provider-dept-8001工程的DeptController

  - 主要是注入了DiscoverClient类，以及新增方法

    ```java
    @RestController
    public class DeptController {
    
        @Autowired
        private DeptService service;
        @Autowired
        private DiscoveryClient client;
    
        @PostMapping(value = "/dept/add")
        public boolean add(@RequestBody Dept dept) {
            return service.add(dept);
        }
    
        @GetMapping(value = "/dept/get/{id}")
        public Dept get(@PathVariable("id") Long id) {
            return service.get(id);
        }
    
        @GetMapping(value = "/dept/list")
        public List<Dept> list() {
            return service.list();
        }
    
        @GetMapping(value = "/dept/discovery")
        public Object discovery() {
            List<String> list = client.getServices();
            System.out.println("**************************" + list);
            List<ServiceInstance> serviceInstanceList = client.getInstances("MICROSERVICECLOUD-DEPT");
            serviceInstanceList.stream().forEach(serviceInstance -> {
                System.out.println(serviceInstance.getServiceId() + "\t" + serviceInstance.getHost() + "\t"
                        + serviceInstance.getPort() + "\t" + serviceInstance.getUri());
            });
            return this.client;
        }
    
    }
    ```

- DeptProvider8001_App主启动类

  - 主要就是加@EnableDiscoverClient注解

    ```java
    @SpringBootApplication
    // 本服务启动后会自动注册进Eureka服务中
    @EnableEurekaClient
    // 服务发现
    @EnableDiscoveryClient
    public class DeptProvider8001_App {
    
        public static void main(String[] args) {
            SpringApplication.run(DeptProvider8001_App.class,args);
        }
    
    }
    ```

- 自测

  - 启动Eureka Server
  - 在启动DeptProvider8001_App主启动类，需要稍等一会
  - 访问http://localhost:8001/dept/discovery

- 修改microservicecloud-consumer-dept-80工程的DeptController_Consumer

  - 新增方法：

    ```java
    /**
     * 测试@EnableDiscoveryClient，消费端可以调用服务发现
     */
    @GetMapping(value = "/consumer/dept/discovery")
    public Object discovery(){
        return restTemplate.getForObject(REST_URL_PREFIX+"/dept/discovery",Object.class);
    }
    ```

  - 完整内容：

    ```java
    /**
     * @author xieweidu
     * @createDate 2019-12-05 22:42
     */
    @RestController
    public class DeptController_Consumer {
    
        private static final String REST_URL_PREFIX = "http://localhost:8001";
    
        @Autowired
        private RestTemplate restTemplate;
    
        @PostMapping("/consumer/dept/add")
        public boolean add(Dept dept){
            return restTemplate.postForObject(REST_URL_PREFIX+"/dept/add",dept,Boolean.class);
        }
    
        @GetMapping("/consumer/dept/get/{id}")
        public Dept get(@PathVariable("id") Long id){
            return restTemplate.getForObject(REST_URL_PREFIX+"/dept/get/"+id,Dept.class);
        }
    
        @GetMapping("/consumer/dept/list")
        public List<Dept> list(){
            return restTemplate.getForObject(REST_URL_PREFIX+"/dept/list",List.class);
        }
    
        /**
         * 测试@EnableDiscoveryClient，消费端可以调用服务发现
         */
        @GetMapping(value = "/consumer/dept/discovery")
        public Object discovery(){
            return restTemplate.getForObject(REST_URL_PREFIX+"/dept/discovery",Object.class);
        }
    
    }
    ```

    

## 集群配置

1. 原理说明

   - 多个服务作为一个整体对外暴露；

2. 新建microservicecloud-eureka-7002以及microservicecloud-eureka-7003项目

3. 按照7001作为模板复制pom文件

   也就是添加以下依赖到7002和7003项目的pom文件中即可：

   ```xml
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
   ```

4. 修改7002和7003的主启动类

   也就是复制7001的主启动类到7002、7003，内容修改为7002、7003相关，如：

   ```java
   @SpringBootApplication
   // EurekaServer服务端启动类，接受其他微服务注册进来
   @EnableEurekaServer
   public class EurekaServer7003_App {
   
       public static void main(String[] args) {
           SpringApplication.run(EurekaServer7003_App.class,args);
       }
   
   }
   ```

5. 修改映射配置

   - 找到C:\Windows\System32\drivers\etc路径下的hosts文件

   - 修改映射配置添加进hosts文件

     ```
     127.0.0.1	eureka7001.com
     127.0.0.1	eureka7002.com
     127.0.0.1	eureka7003.com
     ```

6. 3台eureka服务器的yaml配置

   - **也就是端口号，以及eureka实例的名称需要不一样，以及服务地址为三个**

   - 如：7001配置文件

     ```yaml
     server:
       port: 7001
     eureka:
       instance:
         hostname: eureka7001.com                   # eureka服务端的实例名称
       client:
         register-with-eureka: false                # false表示不向注册中心注册自己
         fetch-registry: false                      # false表示自己端就是注册中心，职责就是维护服务实例，并不需要去检索服务
         service-url:
     #      单机：defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/    # 设置与Eureka Server交互的地址查询服务和注册服务都需要依赖这个地址
           defaultZone: http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/
     ```

7. microservicecloud-provider-dept-8001微服务发布到上面3台eureka集群配置中

   - application.yaml（修改eureka注册的服务地址即可）

     ```yaml
     eureka:
       client:                                                   # 将客户端注册进Eureka
         service-url:
           defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/
       instance:
         instance-id: microservicecloud-dept-8001                # 自定义服务名称信息
         prefer-ip-address: true
     ```



## Eureka遵守的AP原则

Netflix在设计Eureka的时候遵守的就是AP原则；

CAP理论：

- C：Consistency（强一致性）
- A：Availability（可用性）
- P：Partition tolerance（分区容错性）



## 作为服务注册中心，Eureka比Zookeeper好在哪里？

著名的CAP理论指出，一个分布式系统不可能同时满足C（一致性）、A（可用性）和P（分区容错性）。由于分区容错性P在分布式系统中必须是要保证的，所以我们只能在A和C之间进行权衡；

**Eureka保证的是AP，而Zookeeper保证的是CP**；



### Zookeeper保证CP

当向注册中心查询服务列表的时候，我们可以容忍注册中心返回的是几分钟以前的注册信息，但不能接受服务直接down掉不可用。也就是说，服务注册功能对可用性的要求要高于一致性。单zk会出现这么一种情况，当master节点因为网络故障或与其他节点失去联系的时候，剩余节点就会重新进行leader选举。问题在于，选举leader的时间太长，30~120s，且选举期间整个zk集群都是不可用的，这就导致了在选举期间注册服务瘫痪。在云部署的环境下，因网络问题使得zk集群失去master节点是较大概率会发生的情况，虽然服务能够最终恢复，但是漫长的选举时间导致的注册长期不可用是不能容忍的；



## Eureka保证AP

Eureka看明白了这一点，因此在设计的时候就优先保证可用性。**Eureka各个节点都是平等的**，几个节点挂掉并不会影响正常节点的工作，剩余的节点依然可以提供注册和查询服务。而Eureka的客户端在向某个Eureka注册时如果发现连接失败，则会自动切换到其他节点，只要还有一台Eureka存活，就能保证注册服务可用（保证可用性），只不过查到的信息可能不是最新的（不保证强一致性）。除此之外，Eureka还有一种自我保护机制，如果15分钟内超过85%的节点都没有正常的心跳，那么Eureka就认为客户端与注册中心出现了网络故障，此时会出现以下几种情况：

1. Eureka不再从注册列表中移除因为长时间没收到心跳而应该过期的服务；
2. Eureka仍然能够接受新服务的注册和查询请求，但是不会被同步到其他节点上（即保证当前节点依然可用）；
3. 当网络稳定的时候，当前实例新的注册信息会被同步到其他节点中；

**因此，Eureka可以很好的应对因网络故障导致部分节点失去联系的情况，而不会像Zookeeper那样使整个注册服务瘫痪**；















