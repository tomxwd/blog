---
title: 06_SpringCloud_Ribbon负载均衡
date: 2019-12-07 17:51:14
tags: 
 - SpringCloud
categories:
 - SpringCloud
---

# 06_SpringCloud_Ribbon负载均衡

## Ribbon概述

### Ribbon是什么？

SpringCloud Ribbon是基于Netflix Ribbon实现的一套**客户端负载均衡的工具**；

简单来说，Ribbon是Netflix发布的开源项目，主要功能是提供客户端的软件负载均衡算法，将Netflix的中间层服务连接在一起，Ribbon客户端组件提供一系列完整的配置项，如**连接超时、重试**等。简单的说，就是在配置文件中列出Load Balancer（简称LB），后面所有的及其，Ribbon会自动的帮助你基于某种规则（如简单轮询、随机连接等）去连接这些机器。我们也很容易使用Ribbon实现自定义的负载均衡算法；



### Ribbon能干嘛？

LB，即负载均衡（Load Balance），在微服务或分布式集群中经常用到的一种应用；

负载均衡简单的说就是将用户的请求平摊的分配到多个服务上，从而达到系统的HA。

常见的负载均衡软件有Nginx、LVS，硬件F5等，相应的在中间件，例如：dubbo和SpringCloud中均给我们提供了负载均衡，但**SpringCloud的负载均衡算法可以自定义；**



#### 集中式LB

即在服务的消费方和提供方之间使用独立的LB设施（可以是硬件，如F5，也可以是软件，如Nginx），由该设施负责把访问请求通过某种策略转发到服务的提供方；



#### 进程内LB

将LB逻辑集中到消费方，消费方从服务注册中心获知有哪些地址可用，然后自己再从这些地址中选择出一个合适的服务器。

**Ribbon就属于进程内LB**，它只是一个类库，集成于消费方进程，消费方通过它来获取到服务提供方的地址。



### Ribbon官网资料

[Ribbon的github地址](https://github.com/Netflix/ribbon/wiki/Getting-Started)



## Ribbon配置初步

因为Ribbon是客户端负载均衡工具，因此需要在消费端进行修改；

修改microservicecloud-consumer-dept-80工程

- 修改pom.xml文件

  - 修改部分：

    ```xml
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
    
        <artifactId>microservicecloud-consumer-dept-80</artifactId>
        <description>部门微服务消费者</description>
    
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
        </dependencies>
    </project>
    ```

- 修改application.yaml，追加Eureka的服务注册地址

  - 修改部分：

    ```yaml
    eureka:
      client:
        register-with-eureka: false
        service-url:
          defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka,http://eureka7003.com:7003/eureka
    ```

  - 完整内容：

    ```yaml
    server:
      port: 80
    eureka:
      client:
        register-with-eureka: false
        service-url:
          defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka,http://eureka7003.com:7003/eureka
    ```

- 对ConfigBean进行新的注解@LoadBalanced获得RestTemplate的时候加入Ribbon的配置

  - 修改部分：

    在注入RestTemplate的时候加上@LoadBalanced注解即可；

  - 完整内容：

    ```java
    @Configuration
    public class ConfigBean {
    
        @Bean
        @LoadBalanced
        public RestTemplate getRestTemplate(){
            return new RestTemplate();
        }
    
    }
    ```

- 主启动类DeptConsumer80_App添加@EnableEurekaClient注解

  - 修改部分：

    在主启动类上添加@EnableEurekaClient注解，开启Eureka服务端配置；

  - 完整内容：

    ```java
    @SpringBootApplication
    @EnableEurekaClient
    public class DeptConsumer80_App {
    
        public static void main(String[] args) {
            SpringApplication.run(DeptConsumer80_App.class, args);
        }
    
    }
    ```

- 修改DeptController_Consumer客户端访问类

  修改访问服务的基本路径，改为"http://在Eureka注册中心的应用名"

  - 修改部分：

    ```java
    //    private static final String REST_URL_PREFIX = "http://localhost:8001";
    
    private static final String REST_URL_PREFIX = "http://MICROSERVICECLOUD-DEPT";
    ```

  - 完整内容：

    ```java
    @RestController
    public class DeptController_Consumer {
    
    //    private static final String REST_URL_PREFIX = "http://localhost:8001";
    
        private static final String REST_URL_PREFIX = "http://MICROSERVICECLOUD-DEPT";
    
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

- 先启动3个Eureka集群后，再启动microservicecloud-provider-dept-8001并注册进Eureka

- 启动microservicecloud-consumer-dept-80

- 测试

  访问 http://localhost/consumer/dept/list，http://localhost/consumer/dept/get/1

**总结**：

Ribbon和Eureka整合后，Consumer可以直接调用服务而不用关心地址和端口号；



## Ribbon负载均衡

### Ribbon负载均衡架构说明

![image-20191207183445809](06_SpringCloud_Ribbon%E8%B4%9F%E8%BD%BD%E5%9D%87%E8%A1%A1/image-20191207183445809.png)

Ribbon在工作的时候分成两步：

1. 选择Eureka Server，它优先选择在同一个区域内负载均衡较少的Server；
2. 再根据用户指定的策略，再从Server取到的服务注册列表中选择一个地址；

其中，Ribbon提供了多种策略：比如轮询、随机、根据响应时间加权等；



### 构建多个服务提供端

1. 参考microservicecloud-provider-dept-8001，构建8002、8003项目

   - pom.xml
   - src代码
   - resource文件

2. 新建8002、8003数据库，各自微服务分别连各自的数据库

   复制cloudDB01的，然后修改为02,03的数据即可；

3. 修改8002、8003各自的application.yaml配置文件

   - 服务端口号
   - 数据库
   - 对外暴露的统一的服务实例名，spring应用名绝对不能动！【spring.application.name】
   - **eureka.instance.instance-id需要修改！**

   例：

   ```yaml
   server:
     port: 8002
   
   
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
       url: jdbc:mysql://120.25.120.125:12350/cloudDB02?useUnicode=true&characterEncoding=utf8        # 数据库名称
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
         defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/
     instance:
       instance-id: microservicecloud-dept-8002                # 自定义服务名称信息
       prefer-ip-address: true                                 # 访问路径可以显示IP地址
   info:
     app.name: tomxwd-microservicecloud
     company.name: www.tomxwd.top
     build.artifactId: $project.artifactId$
     bulid.version: $project.version$
   ```

4. 启动3个Eureka集群配置

5. 启动3个Dept微服务启动并各自测试通过

   - 访问：http://localhost:8001/dept/list
   - 访问：http://localhost:8002/dept/list
   - 访问：http://localhost:8003/dept/list

6. 启动microservicecloud-consumer-dept-80

7. 客户端通过Ribbon完成负载均衡并访问上一步的Dept微服务

   - 可以通过返回的数据里，db_source一直在变，得知Ribbon负载均衡起作用了；

**总结：**

Ribbon其实就是一个**软负载均衡的客户端组件**，他可以和其他所需请求的客户端结合使用，和Eureka结合只是其中的一个实例；





## Ribbon核心组件IRule

IRule：根据特定算法，从服务列表中选取一个要访问的服务

1. **RoundRobinRule**：轮询
2. **RandomRule**：随机
3. **AvailabilityFilteringRule**：会先过滤掉由于多次访问故障而处于断路器跳闸状态的服务，还有并发的连接数量超过阈值的服务，然后对剩余的服务列表按照轮询策略进行访问；
4. **WeightedResponseTimeRule**：根据平均响应事件计算所有服务的权重，响应时间越快，服务权重越大，被选中的概率越高。刚启动的时候如果统计信息不足，则使用RoundRobinRule策略，等统计信息足够多的时候，会切换到WeightedResponseTimeRule；
5. **RetryRule**：先按照RoundRobinRule的策略获取服务，如果获取服务失败则在指定事件内会进行重试，获取可用的服务；
6. **BestAvailableRule**：会先过滤掉由于多次访问故障而处于断路器跳闸状态的服务，然后选择一个并发量最小的服务；
7. **ZoneAvoidanceRule**：默认规则，复合判断server所在区域的性能和server的可用性选择服务器；

可以停止80工程，然后修改一下代码，改变负载均衡策略；

在ConfigBean文件中增加代码：

```java
@Bean
public IRule myRule(){
    // 用我们重新选择的随机算法替代默认的轮询
    return new RandomRule();
}
```

完整代码：

```java
@Configuration
public class ConfigBean {

    @Bean
    @LoadBalanced
    public RestTemplate getRestTemplate(){
        return new RestTemplate();
    }

    @Bean
    public IRule myRule(){
        // 用我们重新选择的随机算法替代默认的轮询
        return new RandomRule();
    }

}
```





## Ribbon自定义策略

修改microservicecloud-consumer-dept-80

1. 主启动类添加@RibbonClient注解；

   - 在启动微服务的时候就能去加载我们自定义的Ribbon配置类，从而使配置生效，形如：

     ```java
     @RibbonClient(name="MICROSERVICECLOUD-DEPT",configuration=MySelfRule.class)
     ```

2. 注意配置细节：

   **官方文档明确给出了警告：**

   **这个自定义配置类不能放在@ComponentScan所扫描的当前包下以及子包下，否则我们自定义的这个配置类就会被所有的Ribbon客户端所共享，也就是说我们达不到特殊定制化的目的了；**

3. 因此新建一个包，top.tomxwd.ribbon.rule，然后再写MySelfRule类的代码

   ```java
   @Configuration
   public class MySelfRule {
   
       @Bean
       public IRule myRule(){
           // 默认是轮询，自定义为随机
           return new RandomRule();
       }
   
   }
   ```

   注意类上打@Configuration注解，然后注入IRule类返回到容器；

   注意，如果要测试的话，先把ConfigBean中注入的策略改为轮询策略，然后才能看出自定义策略返回的随机策略是否生效；



### 自定义规则深度解析

![继承关系](06_SpringCloud_Ribbon%E8%B4%9F%E8%BD%BD%E5%9D%87%E8%A1%A1/image-20191208104546675.png)

- 问题：依旧轮询策略，但是加上新需求，每个服务器要求被调用5次，也就是说之前是每个机器一次，再到下一个及其，现在是每台机器5次才轮到下一个机器；

- 解析源码：

- 参考源码修改为我们需求要求的FiveTimeRobinRule.java

  ```java
  /**
   * 五次轮询策略
   *
   * @author xieweidu
   * @createDate 2019-12-07 21:38
   */
  public class FiveTimeRobinRule extends AbstractLoadBalancerRule {
  
      /**
       * 当total==5的时候，才往下走一步轮询，然后total置零 currentIndex++，需要注意总共的index有多少
       */
      private AtomicInteger total = new AtomicInteger(0);
      /**
       * 当前对外提供服务的服务器地址
       */
      private AtomicInteger currentIndex = new AtomicInteger(0);
  
      public Server choose(ILoadBalancer lb, Object key) {
          if (lb == null) {
              return null;
          }
          Server server = null;
  
          while (server == null) {
              if (Thread.interrupted()) {
                  return null;
              }
              List<Server> upList = lb.getReachableServers();
              List<Server> allList = lb.getAllServers();
  
              int serverCount = allList.size();
              if (serverCount == 0) {
                  /*
                   * No servers. End regardless of pass, because subsequent passes
                   * only get more restrictive.
                   */
                  return null;
              }
  
  //            int index = rand.nextInt(serverCount);
  //            server = upList.get(index);
              // 主要就是下面这段代码做到了5次轮询
              if (total.get() < 5) {
                  total.incrementAndGet();
                  server = upList.get(currentIndex.get());
              } else {
                  total.set(0);
                  currentIndex.incrementAndGet();
                  if (currentIndex.get() >= upList.size()) {
                      currentIndex.set(0);
                  }
              }
  
              if (server == null) {
                  /*
                   * The only time this should happen is if the server list were
                   * somehow trimmed. This is a transient condition. Retry after
                   * yielding.
                   */
                  Thread.yield();
                  continue;
              }
  
              if (server.isAlive()) {
                  return (server);
              }
  
              // Shouldn't actually happen.. but must be transient or a bug.
              server = null;
              Thread.yield();
          }
  
          return server;
  
      }
  
      @Override
      public Server choose(Object key) {
          return choose(getLoadBalancer(), key);
      }
  
      @Override
      public void initWithNiwsConfig(IClientConfig clientConfig) {
          // TODO Auto-generated method stub
  
      }
  }
  ```

- 改为我们自定义的策略

  ```java
  @Configuration
  public class MySelfRule {
  
      @Bean
      public IRule myRule(){
          // 默认是轮询，自定义为随机
  //        return new RandomRule();
          // 改为自定义的五次轮询
          return new FiveTimeRobinRule();
      }
  
  }
  ```

- 调用

  ```java
  // 在启动微服务的时候就能去加载我们自定义的Ribbon配置类，从而使配置生效
  @RibbonClient(name="MICROSERVICECLOUD-DEPT",configuration= MySelfRule.class)
  ```

- 测试

  访问： http://localhost/consumer/dept/list ，查看db_source的变化即可；



