---
title: 08_SpringCloud_Hystrix断路器
date: 2019-12-08 16:15:22
tags: 
 - SpringCloud
categories:
 - SpringCloud
---

# 08_SpringCloud_Hystrix断路器

## Hystrix概述

### 分布式系统面临的问题

复杂的分布式体系结构中的应用程序有数十个依赖关系，每个依赖关系在某些时候将不可避免的失败；



#### 服务雪崩

多个微服务之间调用的时候，假设微服务A调用微服务B和微服务C，微服务B和微服务C又调用其他的微服务，这就是所谓的**”扇出“**。如果扇出的链路上某个微服务的调用响应时间过长或者不可用，对微服务A的调用就会占用越来越多的系统资源，进而引起系统崩溃，所谓的”雪崩效应“；

对于高流量的应用来说，单一的后端依赖可能会导致所有服务器上的所有资源在几秒内饱和。比失败更糟糕的是，这些应用程序还可能导致服务之间的延迟增加、备份队列、线程和其他系统资源紧张，导致整个系统发生更多的级联故障。这些都表示需要对故障和延迟进行隔离和管理，以便单个依赖关系的失败，不能取消整个应用程序或系统；



### Hystrix是什么

Hystrix是一个用于处理分布式系统的**延迟**和**容错**的开源库，在分布式系统中，许多依赖不可避免的会调用失败，比如超时、异常等，Hystrix能够保证在一个依赖出问题的情况下，**不会导致整个服务失败，避免级联故障，以提高分布式系统的弹性**；

“断路器“本身是一种开关装置，当某个服务单元发生故障之后，通过断路器的故障监控（类似熔断保险丝），**向调用者返回一个符合预期的、可处理的备选响应（FallBack），而不是长时间的等待或抛出调用方法无法处理的异常**，这样就保证了服务调用方的线程不会被长时间、不必要地占用，从而避免了故障在分布式系统中的蔓延，乃至雪崩；



### Hystrix能干嘛

1. 服务降级
2. 服务熔断
3. 服务限流
4. 接近实时的监控
5. ......



### 官网资料

直接找github上的[Hystrix](https://github.com/Netflix/Hystrix/wiki)项目，查看里面的wiki；



## Hystrix服务熔断

### 服务熔断是什么？

熔断机制是应对雪崩效应的一种微服务链路保护机制；

当扇出链路的某个微服务不可用或者响应时间太长的时候，会进行服务的降级，**进而熔断该节点微服务的调用，快速返回“错误”的响应信息**。当检测到该节点微服务调用响应正常后恢复调用链路。在SpringCloud框架里熔断机制通过Hystrix实现。Hystrix会监控微服务调用的状况，当失败的调用达到一定的阈值，缺省是5秒内20次调用失败就会启动熔断机制。熔断机制的注解是@HystrixCommand。

**一般是某个服务故障或者异常引起，类似现实世界中的“保险丝”，当某个异常触发的时候，直接熔断整个服务，而不是一直等到此服务超时**；



### 构建Hystrix的CircuitBreaker进项目

- 参考microservicecloud-provider-dept-8001新建一个新的项目microservicecloud-provider-dept-hystrix-8001；

  - pom.xml

    - 修改内容

      ```xml
      <!-- Hystrix断路器相关依赖 -->
      <dependency>
          <groupId>org.springframework.cloud</groupId>
          <artifactId>spring-cloud-starter-hystrix</artifactId>
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
      
          <artifactId>microservicecloud-provider-dept-hystrix-8001</artifactId>
      
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
              <!-- Hystrix断路器相关依赖 -->
              <dependency>
                  <groupId>org.springframework.cloud</groupId>
                  <artifactId>spring-cloud-starter-hystrix</artifactId>
              </dependency>
          </dependencies>
      </project>
      ```

  - application.yaml

    **主要就是修改eureka.instance.instance-id为microservicecloud-dept8001-hystrix；**服务名称修改；

    - 修改部分：
    
      ```yaml
    eureka:
        client:                                                   # 将客户端注册进Eureka
        service-url:
            defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/
        instance:
          instance-id: microservicecloud-dept8001-hystrix         # 自定义服务名称信息
          prefer-ip-address: true                                 # 访问路径可以显示IP地址
      ```
    
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
          url: jdbc:mysql://localhost:3306?useUnicode=true&characterEncoding=utf8        # 数据库名称
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
          instance-id: microservicecloud-dept8001-hystrix         # 自定义服务名称信息
          prefer-ip-address: true                                 # 访问路径可以显示IP地址
      info:
        app.name: tomxwd-microservicecloud
        company.name: www.tomxwd.top
        build.artifactId: $project.artifactId$
        bulid.version: $project.version$
      ```

- 修改DeptController

  - **@HystrixCommand报异常后如何处理**

    一旦调用服务方法失败并抛出了错误信息之后，会自动调用@HystrixCommand标注好的fallbackMethod调用类中的指定方法；

  - 代码内容

    ```java
    @RestController
    public class DeptController {
    
        @Autowired
        private DeptService service = null;
    
        @GetMapping(value = "/dept/get/{id}")
        // 一旦调用服务方法失败并抛出了错误信息之后，会自动调用@HystrixCommand标注好的fallbackMethod调用类中的指定方法
        @HystrixCommand(fallbackMethod = "processHystrix_Get")
        public Dept get(@PathVariable("id") Long id) {
            Dept dept = this.service.get(id);
            if (null == dept) {
                throw new RuntimeException("该ID: " + id + " 没有找到对应的信息");
            }
            return dept;
        }
    
        public Dept processHystrix_Get(@PathVariable("id") Long id) {
            return new Dept().setDeptno(id).setDname("该ID: " + id + "没有找到对应的信息,null--@HystrixCommand")
                    .setDb_source("no this database in MySQL");
        }
    
    }
    ```

- 修改主启动类名称为DeptProvider8001_Hystrix_App，并**添加新注解@EnableCircuitBreaker**；

  ```java
  @SpringBootApplication
  // 本服务启动后会自动注册进Eureka服务中
  @EnableEurekaClient
  // 服务发现
  @EnableDiscoveryClient
  // 对Hystrix熔断机制的支持
  @EnableCircuitBreaker
  public class DeptProvider8001_Hystrix_App {
  
      public static void main(String[] args) {
          SpringApplication.run(DeptProvider8001_Hystrix_App.class,args);
      }
  
  }
  ```

- 测试

  - 启动Eureka集群
  - 启动主启动类：DeptProvider8001_Hystrix_App
  - Consumer启动microservicecloud-consumer-dept-80或者Feign版的也可以
  - 访问http://localhost/consumer/dept/get/112
    - 正常返回`{"deptno":117,"dname":"该ID: 117没有找到对应的信息,null--@HystrixCommand","db_source":"no this database in MySQL"}`



## Hystrix服务降级

### 服务降级是什么

整体资源快不够了，忍痛将某些服务先关掉，等到难关度过了，再重新开启，比如淘宝双十一的时候把修改地址的服务断了，点击后返回提示信息不允许修改，等到双十一访问压力过了再重新开启修改地址的功能；

**服务的降级处理是在客户端实现完成的，与服务端没有关系**；

**所谓服务降级，一般是从整体符合考虑。就是当某个服务熔断之后，服务器将不再被调用，此时客户端可以自己准备一个本地的fallback回调，返回一个缺省值。这样做，虽然服务水平下降，但是好歹可用，比直接挂掉要强；**



### 构建Hystrix的

- 修改microservicecloud-api工程，根据已有的DeptClientService接口，新建一个实现了FallbackFactory接口的类DeptClientServiceFallbackFactory；

  **千万不要忘记在类上面增加@Component注解！！！！！！**

  ```java
  /**
   * 不要忘记添加@Component注解
   */
  @Component
  public class DeptClientServiceFallbackFactory implements FallbackFactory<DeptClientService> {
  
      @Override
      public DeptClientService create(Throwable throwable) {
          return new DeptClientService() {
              @Override
              public Dept get(long id) {
                  return new Dept().setDeptno(id).setDname("该ID: "+id+"没有找到对应的信息,Consumer客户端提供的降级信息，此刻服务Provider已经关闭")
                          .setDb_source("no this database in MySQL");
              }
  
              @Override
              public List<Dept> list() {
                  return null;
              }
  
              @Override
              public boolean add(Dept dept) {
                  return false;
              }
          };
      }
  }
  ```

- 修改microservicecloud-api工程，DeptClientService接口**在注解@FeignClient中添加fallbackFactory属性值**

  `@FeignClient(value = "MICROSERVICECLOUD-DEPT",fallbackFactory = DeptClientServiceFallbackFactory.class)`

  ```java
  @FeignClient(value = "MICROSERVICECLOUD-DEPT",fallbackFactory = DeptClientServiceFallbackFactory.class)
  public interface DeptClientService {
  
      @GetMapping(value = "/dept/get/{id}")
      Dept get(@PathVariable("id")long id);
  
      @GetMapping(value = "/dept/list")
      List<Dept> list();
  
      @PostMapping(value = "/dept/add")
      public boolean add(Dept dept);
  
  }
  ```

- 修改了api工程，需要mvn clean install一下；

- microservicecloud-consumer-dept-feign工程修改application.yaml配置文件：

  - 修改部分：

    ```yaml
    feign:
      hystrix:
        enabled: true
    ```

  - 完整内容：

    ```yaml
    server:
      port: 80
    
    feign:
      hystrix:
        enabled: true
    eureka:
      client:
        register-with-eureka: false
        service-url:
          defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka,http://eureka7003.com:7003/eureka
    ```

- 测试

  - 启动Eureka集群

  - 启动microservicecloud-provider-dept-8001【注意是不带Hystrix版本的provider】

  - 启动microservicecloud-consumer-dept-feign启动

  - 正常访问测试

    - 访问：http://localhost/consumer/dept/get/1

  - **故意关闭**微服务microservicecloud-provider-dept-8001

    - 访问：http://localhost/consumer/dept/get/1

      返回我们自定义的结果：

      ```json
      {
      	"deptno": 1,
      	"dname": "该ID: 1没有找到对应的信息,Consumer客户端提供的降级信息，此刻服务Provider已经关闭",
      	"db_source": "no this database in MySQL"
      }
      ```

  - 客户端自己调用提示

    - 上一步调用的时候，provider已经down了，但是我们做了服务降级处理，让**客户端在服务端不可用的时候也会获得提示信息而不会挂起耗死服务器；**



## 服务监控Hystrix Dashboard

除了隔离依赖服务的调用之外，Hystrix还提供了**准实时的调用监控（Hystrix Dashboard）**，Hystrix会持续地记录所有通过Hystrix发起的请求的执行信息，并以统计报表和图形的形式展示给用户，包括每秒执行多少请求，多少成功，多少失败等等；

Netflix通过hystrix-metrics-event-stream项目实现了对以上指标的监控。SpringCloud也提供了Hystrix Dashboard的整合，对监控内容转化为可视化界面；



### 整合Hystrix Dashboard

- 新建工程microservicecloud-consumer-hystrix-dashboard

- pom.xml

  添加关于Hystrix以及Hystrix Dashboard的相关依赖；

  - 修改部分：

    ```xml
    <!-- 添加hystrix和hystrix-dashboard相关依赖 -->
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-hystrix</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-hystrix-dashboard</artifactId>
        </dependency>
    </dependencies>
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
    
        <artifactId>microservicecloud-consumer-hystrix-dashboard</artifactId>
    
        <!-- 添加hystrix和hystrix-dashboard相关依赖 -->
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-starter-hystrix</artifactId>
            </dependency>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-starter-hystrix-dashboard</artifactId>
            </dependency>
        </dependencies>
    
    </project>
    ```

- application.yaml

  添加端口信息即可；

  - 修改部分：

    ```yaml
    server:
      port: 9001
    ```

  - 完整内容：

    ```yaml
    server:
      port: 9001
    ```

- 编写主启动类DeptConsumer_DashBoard_App，以及**新增@EnableHystrixDashboard**注解

  ```java
  @SpringBootApplication
  @EnableHystrixDashboard
  public class DeptConsumer_DashBoard_App {
  
      public static void main(String[] args) {
          SpringApplication.run(DeptConsumer_DashBoard_App.class, args);
      }
  
  }
  ```

- **所有Provider微服务提供类（8001/8002/8003）都需要添加监控依赖配置**

  ```xml
  <!-- actuator监控以及信息配置 -->
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-actuator</artifactId>
  </dependency>
  ```

  如果已经添加，就不要再加了

- 启动microservicecloud-consumer-hystrix-dashboard，该微服务负责监控消费端

  - 访问：http://localhost:9001/hystrix

    出现以下页面则为成功：

    ![hystrix-dashboard页面](08_SpringCloud_Hystrix%E6%96%AD%E8%B7%AF%E5%99%A8/image-20191208185206557.png)

    可以看到下面那句， *Single Hystrix App:* http://hystrix-app:port/hystrix.stream；

    也就是说单个应用访问：http://应用名:端口/hystrix.stream

- 启动Eureka集群

- 启动microservicecloud-provider-dept-hystrix-8001

  - 访问：http://localhost:8001/dept/get/1
  - 访问：http://localhost:8001/hystrix.stream
    - 可以看到一直有数据在输出

- 监控测试

  - 多次刷新http://localhost:8001/dept/get/1

  - **观察监控窗口：**

    - 填写监控地址

      - 在Hystrix Dashboard网页上，填写http://localhost:8001/hystrix.stream即可；
      - Delay：该参数用来控制服务器上轮询监控信息的延迟时间，默认2000毫秒，可以通过配置该属性来降低客户端的网络和CPU消耗；
      - Title：该参数对应了头部标题Hystrix Stream之后的内容，默认会使用具体监控实例的URL，可以通过配置该信息来展示更合适的标题；

    - 监控结果

      - 7色

      - 1圈

        - 实心圆：共有两种含义。它通过颜色的变化代表了实例的健康程度，它的健康度从绿色<黄色<橙色<红色递减；

        - 该实心圆除了颜色的变化之外，它的大小也会根据实例的请求流量发生变化，流量越大该实心圆就越大。所以通过该实心圆的展示，就可以在大量的实例中快速的发现**故障实例和高压力实例**；

      - 1线

        - 曲线：用来记录2分钟内流量的相对变化，可以通过它来观察到流量的上升和下降趋势；

      - 整图说明

        ![image-20191208191153245](08_SpringCloud_Hystrix%E6%96%AD%E8%B7%AF%E5%99%A8/image-20191208191153245.png)

        ![image-20191208191252640](08_SpringCloud_Hystrix%E6%96%AD%E8%B7%AF%E5%99%A8/image-20191208191252640.png)

    - 如何看？

      - 如果多个应用，可以看出应用执行情况，方便运维；

    - 搞懂一个才能看懂复杂的监控

      ![image-20191208191632924](08_SpringCloud_Hystrix%E6%96%AD%E8%B7%AF%E5%99%A8/image-20191208191632924.png)















