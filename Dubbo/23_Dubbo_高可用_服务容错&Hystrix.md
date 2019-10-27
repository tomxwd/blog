---
title: 23_Dubbo_高可用_服务容错&Hystrix
date: 2019-09-22 15:03:32
tags: 
 - Dubbo
categories:
 - Java
 - Dubbo
---

# 23_Dubbo\_高可用_服务容错&Hystrix

在集群调用失败的时候，Dubbo提供了多种容错方案，缺省为failover重试



## Dubbo集群容错模式

### Failover Cluster

失败自动切换，当出现失败的时候，重试其他服务器。通常用于读操作，但重试会带来更长的延迟。可通过retries=2来设置重试次数（不包含第一次）

重试次数配置如下：

- `<dubbo:service retries="2" />`

- `<dubbo:reference retries="2"/>`

- ```xml
  <dubbo:reference>
      <dubbo:method name="findFoo" retries="2"></dubbo:method>
  </dubbo:reference>
  ```



### Failfast Cluster

快速失败，只发起一次调用，失败立即报错。通常用于非幂等性的写操作，比如新增记录。



### Failsafe Cluster

失败安全，出现异常的时候，直接忽略。通常用于写入审计日志等操作；



### Failback Cluster

失败自动恢复，后台记录失败请求，定时重发。通常用于消息通知操作；



### Forking Cluster

并行调用多个服务器，只要一个成功即返回。通常用于实时性要求较高的读操作，但需要浪费更多服务资源。可通过forks="2"来设置最大并行数。



### Broadcast Cluster

广播调用所有提供者，逐个调用，任意一台报错则报错，通常用于通知所有提供者更新缓存或日志等本地资源信息。



### 集群模式配置

按照以下示例在服务提供方和消费方配置集群模式。

- `<dubbo:service cluster="failsafe" />`
- `<dubbo:reference cluster="failsafe"/>`



## 整合Hystrix

Hystrix旨在通过控制那些访问远程系统、服务和第三方库的节点，从而对延迟和故障提供更强大的容错能力。Hystrix具备拥有回退机制和断路器功能的线程和信号隔离，请求缓存和请求打包，以及监控和配置等功能。



### 1. 配置spring-cloud-starter-netfilx-hystrix

springboot官方提供了对hystrix的集成，直接在pom.xml里加入依赖：

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
    <artifactId>spring-clound-starter-netflix-hystrix</artifactId>
    <version>1.4.4.RELEASE</version>
</dependency>
```

这里最好直接生成项目的时候选择，根据当前的springboot版本，我这里完整的配置文件是：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.8.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>top.tomxwd.gmall</groupId>
    <artifactId>boot-user-service-provider</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>boot-user-service-provider</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Greenwich.SR3</spring-cloud.version>
    </properties>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

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

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

然后在Application类上增加@EnableHystrix来启用hystrix starter：

```java
@SpringBootApplication
@EnableHystrix
public class ProviderApplication{
  ......  
}
```

### 2. 配置provider端

在Dubbo的Provider上增加@HystrixCommand配置，这样子调用就会经过Hystrix代理。

```java
@Service
@org.springframework.stereotype.Service
public class UserServiceImpl implements UserService {

    @Override
    @HystrixCommand
    public List<UserAddress> getUserAddressList(String userId) {
        System.out.println("UserServiceImpl.....old...");
        // TODO Auto-generated method stub
        UserAddress address1 = new UserAddress(1, "北京市昌平区宏福科技园综合楼3层", "1", "李老师", "010-56253825", "Y");
        UserAddress address2 = new UserAddress(2, "深圳市宝安区西部硅谷大厦B座3层（深圳分校）", "1", "王老师", "010-56253825", "N");

        if(Math.random()>0.5){
            throw new RuntimeException();
        }
        return Arrays.asList(address1,address2);
    }

}
```

调用的时候会有一半的几率抛出运行时异常。

## 3. 配置consumer端

在调用的方法上也加上@HystrixCommand注解，如下：

```java
@Service
public class OrderServiceImpl implements OrderService {

    @Reference(stub = "top.tomxwd.gmall.service.impl.UserServiceStub")
    UserService userService;

    @Override
    @HystrixCommand(fallbackMethod = "hello")
    public List<UserAddress> initOrder(String userId) {
        // 1、查询用户的收货地址
        List<UserAddress> addressList = userService.getUserAddressList(userId);
        System.out.println("addressList = " + addressList);
        return addressList;
    }

    public List<UserAddress> hello(String userId) {
        return Arrays.asList(new UserAddress(0,"没地址","0","","",""));
    }

}
```

hello指回调的函数名，当调用失败的时候会回调这个函数。