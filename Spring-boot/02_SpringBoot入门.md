---
title: 02_SpringBoot入门
date: 2019-06-05 ‏‎‏‎23:00:02
tags: 
 - Java
 - SpringBoot
categories:
 - Spring
 - SpringBoot初学
---

# 02_SpringBoot入门

## 1、简介
> 简化Spring应用开发的一个框架。
> 整个Spring技术栈的一个大整合。
> J2EE开发的一站式解决方案。

 - SpringBoot来简化Spring应用开发，约定大于配置，去繁从简，just run就能创建一个独立的，产品级别的应用。

   

 - 优点：
  1. **快速**创建独立运行的Spring项目以及与主流框架的**集成**。
  2. 使用嵌入的Servlet容器，**应用无需打成war包**。
  3. **starters自动依赖与版本控制**。
  4. 大量的**自动配置**，简化开发，也可以修改默认值。
  5. 无序配置xml文件，无代码生成，开箱即用。
  6. 准生产环境的运行时应用监控。
  7. 与云计算的天然集成。
 - 缺点：
  1. 入门容易，精通难。
  2. 需要掌握spring技术。



---

## 2、微服务

> 2014年，martin fowler在博客中提出。
> 微服务是一种**架构风格**
> 一个应用应该是一组小型服务。可以通过HTTP的方式进行互通。

**单体应用**：

一个单体应用程序把他所有的功能放在一个单一进程中。。。并且通过在多个服务器上复制这个单体进行扩展

**微服务**：

一个微服务架构把每个功能元素放进一个独立的服务中。。。并且通过跨服务器分发这些服务进行扩展，只在需要时才复制。

每一个功能元素最终都是一个**可独立替换**和**独立升级**的软件单元。
详细参照[微服务提出者的网站](https://martinfowler.com/microservices/)。



---

## 3、环境准备

掌握以下内容：

- Spring框架的使用经验
- 熟练使用Maven进行项目构建和管理依赖
- 熟练使用Eclipse或者IDEA

环境约束：

- JDK1.8
- maven3.X
- IDEA 2017
- Spring Boot 1.5.9RELEASE



---

## 4、SpringBoot HelloWorld

1. 创建Maven工程（jar的形式）

2. 导入springboot相关依赖

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
	<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
       <modelVersion>4.0.0</modelVersion>
   
       <groupId>org.springframework</groupId>
       <artifactId>gs-spring-boot</artifactId>
       <version>0.1.0</version>
   
       <parent>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-parent</artifactId>
           <version>2.1.4.RELEASE</version>
       </parent>
   
       <dependencies>
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-web</artifactId>
           </dependency>
       </dependencies>
   
       <properties>
           <java.version>1.8</java.version>
       </properties>
   
   	<!--可以将应用打包成可执行的jar包-->
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

3. 编写一个主程序

   ```java
   @SpringBootApplication // 标注主程序类，说明是一个SpringBoot应用
   public class Application {
   
       public static void main(String[] args) {
           // 启动Spring应用
           SpringApplication.run(Application.class,args);
       }
   
   }
   ```

   

4. 编写相关的Controller、Service等。 

   这里只给出Controller。Service返回一个字符串即可。

   ```java
   @Controller
   public class HelloController {
   
       @Autowired
       private HelloService service;
   
       @RequestMapping("/hello")
       @ResponseBody
       public String hello(){
           return service.sayhello();
       }
   
   }
   ```

5. 运行：

   只需要运行主类（Application）即可。

6. 简化部署工作

   只需要打Jar包，然后用java -jar 指定jar包，即可访问项目。



---

## 5、原理分析

1. pom文件

   - 父项目

     ```xml
     <parent>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter-parent</artifactId>
     	<version>2.1.4.RELEASE</version>
     </parent>
     ```

     这个父项目还依赖了一个父项目：

     ```xml
     <parent>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-dependencies</artifactId>
         <version>2.1.4.RELEASE</version>
         <relativePath>../../spring-boot-dependencies</relativePath>
     </parent>
     ```

     这个父项目真正管理SpringBoot应用里面所有的依赖版本。

     **SpringBoot的版本仲裁中心**；

     以后我们导入依赖默认是不需要写版本的；

     (没有在dependencies里面管理的依赖自然需要声明版本号)

   - 启动器

     ```xml
     <dependencies>
         <dependency>
             <groupId>org.springframework.boot</groupId>
             <artifactId>spring-boot-starter-web</artifactId>
         </dependency>
     </dependencies>
     ```

     **spring-boot-starter-web**:spring-boot场景启动器；帮我们导入了web模块正常运行所依赖的组件。

     **通过导入各种starter（启动器）来适应所有功能场景。**

2. 主程序类、主入口类

   ```java
   @SpringBootApplication // 标注主程序类，说明是一个SpringBoot应用
   public class Application {
   
       public static void main(String[] args) {
           // 启动Spring应用
           SpringApplication.run(Application.class,args);
       }
   
   }
   ```

   **@SpringBootApplication :** 标注在某个类上说明这个类是SpringBoot的主配置类，SpringBoot就应该运行这个类的main方法来启动SpringBoot应用。

   ```java
   @Target(ElementType.TYPE)
   @Retention(RetentionPolicy.RUNTIME)
   @Documented
   @Inherited
   @SpringBootConfiguration
   @EnableAutoConfiguration
   @ComponentScan(excludeFilters = {
   		@Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
   		@Filter(type = FilterType.CUSTOM,
   				classes = AutoConfigurationExcludeFilter.class) })
   public @interface SpringBootApplication {
   ```

   **@SpringBootConfiguration:**SpringBoot的配置类；标注在某个类上，表示这是个SpringBoot的配置类；

   ```java
   @Target(ElementType.TYPE)
   @Retention(RetentionPolicy.RUNTIME)
   @Documented
   @Configuration
   public @interface SpringBootConfiguration {
   ```

   点进去**@SpringBootConfiguration**发现，@SpringBootConfiguration上面有个**@Configuration**注解，配置类上使用的注解，配置类--->配置文件；

   ```java
   @Target(ElementType.TYPE)
   @Retention(RetentionPolicy.RUNTIME)
   @Documented
   @Component
   public @interface Configuration {
   ```

   再点击**@Configuration**进去发现，他有一个**@Component**注解，配置类也是容器中的一个组件；

   

   再看第二个注解：**@EnableAutoConfiguration**:开启自动配置功能；以前需要我们自己去配置的东西，SpringBoot帮我们自动配置；这个注解告诉SpringBoot开启自动配置，只有写了这个注解，自动配置才会生效。

   ```java
   @Target(ElementType.TYPE)
   @Retention(RetentionPolicy.RUNTIME)
   @Documented
   @Inherited
   @AutoConfigurationPackage
   @Import(AutoConfigurationImportSelector.class)
   public @interface EnableAutoConfiguration {
   ```

   点进去里面

   **@AutoConfigurationPackage**:自动配置包，点进去发现有个**@Import(AutoConfigurationPackages.Registrar.class)**：Spring底层注解@Import，给容器中导入一个组件AutoConfigurationPackages.Registrar.class

   @AutoConfigurationPackage这个注解的作用是将主配置类（@SpringBootApplication标注的类）的所在的包及下面的所有子包下所有的组件扫描到Spring容器。

   而这个注解下还有一个**@Import(AutoConfigurationImportSelector.class)**注解，导入哪些组件的选择器，会将所有需要导入的组件以全限定名的方式返回，这些组件就会被扫描进Spring容器。最终会给容器导入非常多的自动配置类（XxxAutoConfiguration），作用是给容器中导入这个场景需要的所有组件，并配置好这些组件。

   SpringFactoryLoader.loadFactoryNames(EnableAutoConfiguration.class,classLoader)：SpringBoot在启动的时候，会从从类路径下的META-INF/spring.factories中获取EnableAutoConfiguration指定的值，将这些值作为自动配置类导入到容器中，自动配置类就会生效，帮我们进行自动配置工作。以前我们需要自己配置的东西，自动配置类都帮我们做了。

   J2EE的整体解决方案和自动配置都在spring-boot-autoconfigure-1.5.9RELEASE.jar中；

   

---

## 6、使用Spring Initializer快速创建SpringBoot项目

RESTAPI的方式：可以在类上打@Controller以及@ResponseBody，但是可以用**@RestController**来替换；

默认生成的SpringBoot项目有几个特点：

- 主程序已经生成好了，我们只需要编写我们自己的逻辑
- resources文件夹目录结构：
  - static：保存所有的静态资源：js css images;
  
  - templates：保存所有的模板页面：（S pringBoot默认jar包使用嵌入式的Tomcat，默认不支持JSP页面），可以使用模板引擎（freemarker、thymeleaf）；
  
  - application.properties：SpringBoot应用的配置文件 -> 可以用来修改默认配置信息。
  
    ```properties
    server.port=80
    ```
  
    