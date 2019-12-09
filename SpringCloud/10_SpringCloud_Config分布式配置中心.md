---
title: 10_SpringCloud_Config分布式配置中心
date: 2019-12-08 21:13:32
tags: 
 - SpringCloud
categories:
 - SpringCloud
---

# 10_SpringCloud_Config分布式配置中心

## SpringCloud Config配置中心概述

### 分布式系统面临的配置问题

微服务意味着要将单个应用中的业务拆分成一个一个子服务，每个服务的粒度相对较小，因此系统中会出现大量的服务。由于每个服务都需要必要的配置信息才能运行，所以一套集中式的、动态的配置管理设施是必不可少的。

SpringCloud提供了Config Server来解决这个问题，我们每一个微服务自己带着一个application.yaml，上百个配置文件的管理......



### SpringCloud Config是什么？

![image-20191208212304136](10_SpringCloud_Config%E5%88%86%E5%B8%83%E5%BC%8F%E9%85%8D%E7%BD%AE%E4%B8%AD%E5%BF%83/image-20191208212304136.png)

SpringCloud Config为微服务架构中的微服务提供集中化的**外部配置支持**，配置服务器为**各个不同的微服务应用**的所有环境提供了一个**中心化的外部配置**；



### SpringCloud Config怎么玩？

SpringCloud Config分为**服务端和客户端两部分**；

- 服务端：

  服务端也称为**分布式配置中心，它是一个独立的微服务应用**，用于连接配置服务器并且为客户端提供获取配置信息，加密/解密信息等访问接口；

- 客户端：

  客户端则通过指定的配置中心来管理应用资源，以及业务相关的配置内容，并在启动的时候从配置中心获取和加载配置信息；

  配置服务器默认采用git来存储配置信息，这样就有助于对环境配置进行版本管理，并且可以通过git客户端工具来方便的管理和访问配置内容；





### SpringCloud Config能干嘛？

1. 集中管理配置文件

2. 不同环境不同配置，动态化的配置更新，分环境部署，比如dev/test/prod/beta/release
3. 运行期间动态调整配置，不再需要在每个服务部署的机器上编写配置文件，服务会向配置中心统一拉取配置自己的信息
4. 当配置发生变动的时候，服务不需要重新启动即可感知配置的变化并应用新的配置
5. 将配置信息已REST接口的形式暴露



### SpringCloud Config与GitHub整合配置

由于SpringCloud Config默认使用的是Git来存储配置文件（也有其他的方式，比如支持SVN和本地文件），但最推荐的还是Git，而且使用的是http/https访问的形式；



## SpringCloud Config服务端配置

1. 用自己的GitHub账号在GitHub上新建一个名为microservicecloud-config的新仓库

2. 由上一步获得SSH协议的git地址在本地硬盘目录上新建git仓库并clone

   - `git clone git@github.com:tomxwd/microservicecloud-config.git`

3. 在本地的microservicecloud-config里面新建一个application.yaml

   **必须要用UTF-8的形式保存！！！**

   ```yaml
   # 切记保存为UTF-8形式
   spring:
     profiles:
       active:
         -dev
   ---
   spring:
     profiles: dev     # 开发环境
     application:
       name: microservicecloud-config-tomxwd-dev
   ---
   spring:
     profiles: test    # 测试环境
     application:
       name: microservicecloud-config-tomxwd-test
   ```

4. 将上一步的yaml文件推送到github上

   `git add .`

   `git commit -m "提交配置文件"`

   `git push origin master`

5. 新建Module模块microservicecloud-config-3344，它就是Cloud的配置中心模块

6. pom.xml

   - 修改部分：

     ```xml
     <dependency>
         <groupId>org.springframework.cloud</groupId>
         <artifactId>spring-cloud-config-server</artifactId>
     </dependency>
     ```

     如果报错就加：

     ```xml
     <!-- 报错就加这个依赖 -->
     <dependency>
         <groupId>org.eclipse.jgit</groupId>
         <artifactId>org.eclipse.jgit</artifactId>
         <version>4.6.0.201612231935-r</version>
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
     
         <artifactId>microservicecloud-config-3344</artifactId>
     
         <dependencies>
             <!-- springcloud-config -->
             <dependency>
                 <groupId>org.springframework.cloud</groupId>
                 <artifactId>spring-cloud-config-server</artifactId>
             </dependency>
             <!-- 图形化监控 -->
             <dependency>
                 <groupId>org.springframework.boot</groupId>
                 <artifactId>spring-boot-starter-actuator</artifactId>
             </dependency>
             <!-- 熔断 -->
             <dependency>
                 <groupId>org.springframework.cloud</groupId>
                 <artifactId>spring-cloud-starter-hystrix</artifactId>
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

7. application.yaml

   ```yaml
   server:
     port: 3344
   spring:
     application:
       name: microservicecloud-config
     cloud:
       config:
         server:
           git:
             uri: git@github.com:tomxwd/microservicecloud-config.git
   ```

8. 主启动类Config_3344_StartSpringCloudApp

   ```java
   @SpringBootApplication
   @EnableConfigServer
   public class Config_3344_StartSpringCloudApp {
   
       public static void main(String[] args) {
           SpringApplication.run(Config_3344_StartSpringCloudApp.class,args);
       }
   
   }
   ```

9. windows下修改hosts文件，增加映射

   - `127.0.0.1	config-3344.com`

10. 测试通过Config微服务是否可以从GitHub上获取配置内容

    - 启动微服务3344
    - 访问：http://config-3344.com:3344/application-dev.yaml
    - 访问：http://config-3344.com:3344/application-test.yaml
    - 访问：http://config-3344.com:3344/application-xxx.yaml

11. 配置读取规则

12. 成功实现了用SpringCloud Config通过GitHub的方式获取配置信息



## SpringCloud Config客户端配置与测试



## SpringCloud Config配置实战











