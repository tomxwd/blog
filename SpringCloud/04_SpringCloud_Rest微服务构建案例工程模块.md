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

4. microservicecloud-consumer-dept-80部门微服务消费者Module





















