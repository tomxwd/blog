---
title: 07_SpringCloud_Feign负载均衡
date: 2019-12-08 11:19:19
tags: 
 - SpringCloud
categories:
 - SpringCloud
---

# 07_SpringCloud_Feign负载均衡

## Feign概述

- Feign是一个声明式WebService客户端。

- 使用Feign能让编写Web Service客户端更加简单，它的使用方法是定义一个接口，然后在上面添加注解，同时也支持JAX-RS标准的注解。
- Feign也支持可拔插式的编码器和解码器。
- SpringCloud对Feign进行了封装，使其支持了SpringMVC标准注解和HttpMessageConverters。
- Feign可以与Eureka和Ribbon组合使用以支持负载均衡；

**总结：**Feign是一个声明式的Web服务客户端，使得编写Web服务客户端变得非常容易，**只需要创建一个接口，然后在上面添加注解即可；**



## Feign能干嘛？

Feign旨在编写Java Http客户端变得更加容易；

前面在使用Ribbon+RestTemplate的时候，利用RestTemplate对Http请求的封装处理，形成了一套模板化的调用方法。但是在实际开发中，由于对服务依赖的调用可能不止一处，**往往一个接口会被多个地方调用，所以通常都会针对每个微服务自行封装一些客户端类来包装这些依赖服务的调用**。所以，Feign在此基础上做了进一步封装，由他来帮助我们定义和实现依赖服务接口的定义。在Feign的实现下，**我们只需要创建一个接口并使用注解的方式来配置它（以前是Dao接口上面标注Mapper注解，现在是一个微服务接口上面标注一个Feign注解即可）**，即可完成对服务提供方的接口绑定，简化了使用SpringCloud Ribbon时，自动封装服务调用客户端的开发量；



**Feign集成了Ribbon**

利用Ribbon维护了MicroServiceCloud-Dept的服务列表信息，并且通过轮询查询实现了客户端的负载均衡。而与Ribbon不同的是，**通过Feign只需要定义服务绑定接口且以声明式的方法**，优雅而简单的实现了服务调用；



## Feign使用步骤

- 参考microservicecloud-consumer-dept-80，新建一个新的项目——microservicecloud-consumer-dept-feign

  - pom.xml文件

  - application.yaml文件

  - src目录下的java文件

  - 修改主启动类的名字为DeptConsumer80_Feign_App

  - **可以去掉Ribbon相关的配置，如果保留，则也生效**

    - ```java
      @RibbonClient(name="MICROSERVICECLOUD-DEPT",configuration= MySelfRule.class)
      ```

    - 也可以删除ribbon相关的配置类等；

- microservicecloud-consumer-dept-feign工程pom.xml修改，主要添加对feign的支持

  - 修改部分：

    ```xml
    <!-- Feign相关依赖 -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-feign</artifactId>
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
    
        <artifactId>microservicecloud-consumer-dept-feign</artifactId>
    
        <dependencies>
            <dependency>
                <groupId>top.tomxwd</groupId>
                <artifactId>microservicecloud-api</artifactId>
                <version>${project.version}</version>
            </dependency>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-web</artifactId>
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
            <!-- Ribbon相关依赖 -->
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-starter-eureka</artifactId>
            </dependency>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-starter-ribbon</artifactId>
            </dependency>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-starter-config</artifactId>
            </dependency>
            <!-- Feign相关依赖 -->
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-starter-feign</artifactId>
            </dependency>
        </dependencies>
    </project>
    ```

- 修改microservicecloud-api工程

  - 修改pom.xml文件，添加Feign依赖

    ```xml
    <!-- Feign相关依赖 -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-feign</artifactId>
    </dependency>
    ```

  - 新建DeptClientService接口并新增注解@FeignClient

    ```java
    @FeignClient(value = "MICROSERVICECLOUD-DEPT")
    public interface DeptClientService {
    
        @GetMapping(value = "/dept/get/{id}")
        Dept get(@PathVariable("id")long id);
    
        @GetMapping(value = "/dept/list")
        List<Dept> list();
    
        @PostMapping(value = "/dept/add")
        public boolean add(Dept dept);
    
    }
    ```

  - mvn clean install

- microservicecloud-consumer-dept-feign工程修改Controller，添加上一步新建的DeptClientService接口

  ```java
  @RestController
  public class DeptController_Consumer {
  
      @Autowired
      private DeptClientService service = null;
  
      @GetMapping(value = "/consumer/dept/get/{id}")
      public Dept get(@PathVariable("id") Long id){
          return this.service.get(id);
      }
  
      @GetMapping(value = "/consumer/dept/list")
      public List<Dept> list(){
          return this.service.list();
      }
  
      @PostMapping(value = "/consumer/dept/add")
      public Object add(Dept dept){
          return this.service.add(dept);
      }
  
  }
  ```

  **注意注入的Service需要默认为null！**

- microservicecloud-consumer-dept-feign工程修改主启动类配置，加入开启Feign功能支持的注解：

  ```java
  @SpringBootApplication
  @EnableEurekaClient
  // 在启动微服务的时候就能去加载我们自定义的Ribbon配置类，从而使配置生效
  @RibbonClient(name="MICROSERVICECLOUD-DEPT",configuration= MySelfRule.class)
  @EnableFeignClients(basePackages = {"top.tomxwd.springcloud"})
  public class DeptConsumer80_Feign_App {
  
      public static void main(String[] args) {
          SpringApplication.run(DeptConsumer80_Feign_App.class, args);
      }
  
  }
  ```

- 测试

  - 启动3个Eureka集群
  - 启动3个部门微服务8001/8002/8003
  - 启动Feign
  - 访问http://localhost/consumer/dept/list
  - **Feign自带负载均衡配置项**

**总结：**

- **Feign通过接口的方法调用Rest服务（之前是Ribbon+RestTemplate）**
- 请求发送给Eureka服务器（http://MICROSERVICECLOUD-DEPT/dept/list）
- 通过Feign直接找到服务接口，由于在进行服务调用的时候融合了Ribbon技术，所以也支持负载均衡作用；

