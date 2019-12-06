---
title: 04_SpringCloud_Rest微服务构建案例工程模块
date: 2019-12-04 21:44:52
tags: 
 - SpringCloud
categories:
 - SpringCloud
---

# 04_SpringCloud_Rest微服务构建案例工程模块

## 总体介绍

- 以Dept部门模块做一个微服务通用案例Consumer消费者（Client）通过REST调用Provider提供者（Server）提供的服务；

- Maven的分包分模块架构复习
  - SpringCloud父工程（Project）下初次带着3个子模块（Module）
    - microservicecloud-api
      - 封装整体entity/接口/公共配置
    - microservicecloud-provider-dept-8001
      - 服务提供者
      - 8001端口
    - microservicecloud-consumer-dept-80
      - 微服务调用的客户端使用
      - 80端口



## 本次学习SpringCloud版本

SpringCloud选择Dalston.SR1，SpringBoot选择1.5.9.RELEASE



## 构建步骤

1. microservicecloud整体父工程Project

   - 新建父工程microservicecloud，切记Packageing是pom模式

   - 主要是定义POM文件，将后续各个子模块公用的jar包等统一提取出来管理

   - pom文件：

     ```xml
     <?xml version="1.0" encoding="UTF-8"?>
     <project xmlns="http://maven.apache.org/POM/4.0.0"
              xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
              xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
         <modelVersion>4.0.0</modelVersion>
     
         <groupId>top.tomxwd</groupId>
         <artifactId>microservicecloud</artifactId>
         <version>1.0-SNAPSHOT</version>
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
     </project>
     ```

2. microservicecloud-api公共子模块Module

   - 新建microservicecloud-api

     - 创建完成后查看父工程查看pom文件变化

       多了子模块声明：

       ```xml
       <modules>
           <module>microservicecloud-api</module>
       </modules>
       ```

   - 修改POM

     ```xml
     <?xml version="1.0" encoding="UTF-8"?>
     <project xmlns="http://maven.apache.org/POM/4.0.0"
              xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
              xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
         <parent>
             <!-- 子类里面显示声明才能有明确的继承表现，无意外就是父类的默认版本否则自己定义 -->
             <artifactId>microservicecloud</artifactId>
             <groupId>top.tomxwd</groupId>
             <version>1.0-SNAPSHOT</version>
         </parent>
         <modelVersion>4.0.0</modelVersion>
     
         <artifactId>microservicecloud-api</artifactId>
     
         <!-- 当前Module需要用到的jar，如果父工程已经包含版本控制，可以不写版本号信息 -->
         <dependencies>
             <dependency>
                 <groupId>org.projectlombok</groupId>
                 <artifactId>lombok</artifactId>
             </dependency>
         </dependencies>
     
     </project>
     ```

   - 新建部门entity且配合lombok使用

     ```java
     /**
      * 必须序列化
      * @author xieweidu
      * @createDate 2019-12-04 22:52
      */
     @Data
     @AllArgsConstructor
     @NoArgsConstructor
     @Accessors(chain = true)
     public class Dept implements Serializable {
     
         /** 主键 */
         private Long deptno;
         /** 部门名称 */
         private String dname;
         /** 来自哪个数据库，因为微服务架构可以一个服务对应一个数据库，同一个信息被存储到不同的数据库 */
         private String db_source;
     
         /**
          * 仅传入部门名的构造函数
          * @param dname
          */
         public Dept(String dname) {
             this.dname = dname;
         }
     }
     ```

   - mvn clean install后给其他模块引用，达到通用目的。也即需要用到部门实体的话，不用每个工程都定义一份，直接引用本模块即可；

3. microservicecloud-provider-dept-8001部门微服务提供者Module

   - 新建microservicecloud-provider-dept-8001

     - 创建完后查看父工程pom文件变化，发现module多了这个项目

   - pom文件：

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
         </dependencies>
     </project>
     ```

   - yaml文件：

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
         url: jdbc:mysql://loaclhost:3306/cloudDB01?useUnicode=true&characterEncoding=utf8        # 数据库名称
         username: root
         password: root
         dbcp2:
           min-idle: 5                                           # 数据库连接池的最小维持连接数
           initial-size: 5                                       # 初始化连接数
           max-total: 5                                          # 最大连接数
           max-wait-millis: 200                                  # 等待连接获取的最大超时时间
     ```

   - 工程src/main/resources目录下新建mybatis文件夹后新建mybatis.cfg.xml文件

     ```xml
     <?xml version="1.0" encoding="UTF-8" ?>
     <!DOCTYPE configuration
             PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
             "http://mybatis.org/dtd/mybatis-3-config.dtd">
     <configuration>
     
         <settings>
             <!-- 开启二级缓存 -->
             <setting name="cacheEnabled" value="true"/>
         </settings>
     
     </configuration>
     ```

   - MySQL创建部门数据库脚本

     - docker pull mysql:5
     - docker run -itd --name my-mysql -p 12350:3306 -e MYSQL_ROOT_PASSWORD=root mysql:5 --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci

     ```mysql
     SET FOREIGN_KEY_CHECKS=0;
     
     -- ----------------------------
     -- Table structure for `dept`
     -- ----------------------------
     DROP TABLE IF EXISTS `dept`;
     CREATE TABLE `dept` (
       `deptno` bigint(20) NOT NULL AUTO_INCREMENT,
       `dname` varchar(60) COLLATE utf8mb4_unicode_ci DEFAULT NULL,
       `db_source` varchar(60) COLLATE utf8mb4_unicode_ci DEFAULT NULL,
       PRIMARY KEY (`deptno`)
     ) ENGINE=InnoDB AUTO_INCREMENT=6 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
     
     -- ----------------------------
     -- Records of dept
     -- ----------------------------
     INSERT INTO `dept` VALUES ('1', '开发部', 'cloudDB01');
     INSERT INTO `dept` VALUES ('2', '人事部', 'cloudDB01');
     INSERT INTO `dept` VALUES ('3', '财务部', 'cloudDB01');
     INSERT INTO `dept` VALUES ('4', '市场部', 'cloudDB01');
     INSERT INTO `dept` VALUES ('5', '运维部', 'cloudDB01');
     
     ```

   - DeptDao部门接口

     ```java
     @Mapper
     public interface DeptDao {
     
         boolean addDept(Dept dept);
     
         Dept findById(Long id);
     
         List<Dept> findAll();
     
     }
     ```

   - 工程src/main/resources/mybatis目录下新建mapper文件夹后再建DeptMapper.xml

     ```xml
     <?xml version="1.0" encoding="UTF-8" ?>
     <!DOCTYPE mapper
             PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
             "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
     <mapper namespace="top.tomxwd.springcloud.dao.DeptDao">
     
     
         <insert id="addDept">
             INSERT INTO dept(dname,db_source) VALUES (#{dname},DATABASE())
         </insert>
         <select id="findById" resultType="top.tomxwd.springcloud.entity.Dept">
             select deptno,dname,db_source from dept where deptno=#{deptno}
         </select>
         <select id="findAll" resultType="top.tomxwd.springcloud.entity.Dept">
             select deptno,dname,db_source from dept
         </select>
     </mapper>
     ```

   - DeptService部门服务接口

     ```java
     public interface DeptService {
     
         boolean add(Dept dept);
     
         Dept get(Long id);
     
         List<Dept> list();
     
     }
     ```

   - DeptServiceImpl部门服务接口实现类

     ```java
     @Service
     public class DeptServiceImpl implements DeptService {
     
         @Autowired
         private DeptDao dao;
     
         @Override
         public boolean add(Dept dept) {
             return dao.addDept(dept);
         }
     
         @Override
         public Dept get(Long id) {
             return dao.findById(id);
         }
     
         @Override
         public List<Dept> list() {
             return dao.findAll();
         }
     }
     ```

   - DeptController部门微服务提供者REST

     ```java
     @RestController
     public class DeptController {
     
         @Autowired
         private DeptService service;
     
         @PostMapping(value = "/dept/add")
         public boolean add(@RequestBody Dept dept){
             return service.add(dept);
         }
     
         @GetMapping(value = "/dept/get/{id}")
         public Dept get(@PathVariable("id") Long id){
             return service.get(id);
         }
     
         @GetMapping(value = "/dept/list")
         public List<Dept> list(){
             return service.list();
         }
     
     }
     ```

   - DeptProvider8001_App主启动类DeptProvider8001_App

     ```java
     @SpringBootApplication
     public class DeptProvider8001_App {
     
         public static void main(String[] args) {
             SpringApplication.run(DeptProvider8001_App.class,args);
         }
     
     }
     ```

   - 测试

     - 访问 http://localhost:8001/dept/get/1 
     -  http://localhost:8001/dept/list

4. microservicecloud-consumer-dept-80部门微服务消费者Module

   - 新建microservicecloud-consumer-dept-80

   - pom文件：

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
         </dependencies>
     </project>
     ```

   - yaml配置文件

     ```yaml
     server:
       port: 80
     ```

   - top.tomxwd.springcloud.config包下ConfigBean的编写（类似于Spring里面的applicationContext.xml写入的注入bean）也就是配置类；

     - 注入RestTemplate类

     ```java
     @Configuration
     public class ConfigBean {
     
         @Bean
         public RestTemplate getRestTemplate(){
             return new RestTemplate();
         }
     
     }
     ```

   - top.tomxwd.springcloud.controller包下新建DeptController_Consumer部门微服务消费者REST

     - **RestTemplate**
       - 说明：RestTemplate提供了多种便捷访问远程Http服务的方法，是一种简单便捷的访问restful服务模板类，是Spring提供的用于访问Rest服务的客户端模板工具集；
       - 使用：使用RestTemplate访问restful接口非常的简单粗暴无脑。（url、requestMap、ResponseBean.class）这三个参数分别代表REST请求地址、请求参数、HTTP响应转换被转换成的对象类型。

     ```java
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
     
     }
     ```

   - DeptConsumer80_App主启动类

     ```java
     @SpringBootApplication
     public class DeptConsumer80_App {
     
         public static void main(String[] args) {
             SpringApplication.run(DeptConsumer80_App.class, args);
         }
     
     }
     ```

   - 测试

     - 访问 http://localhost/consumer/dept/get/1 
     -  http://localhost/consumer/dept/list 

5. 





















