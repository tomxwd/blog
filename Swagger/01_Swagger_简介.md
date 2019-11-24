---
title: 01_Swagger_简介
date: 2019-11-24 22:06:37
tags: 
 - Swagger
categories:
 - Swagger
---

# 01_Swagger_简介

> [Swagger官网](https://swagger.io/)

- 了解Swagger的作用和概念
- 了解前后端分离
- 在SpringBoot中集成Swagger



## 前后端分离

Vue+SpringBoot



后端时代：

- 前端只用管理静态页面；
- 前端写好的.html文件交给后端；
- 模板引擎JSP=》后端是主力；

前后端分离时代：

- 后端：后端控制层、服务层、数据访问层；
- 前端：前端控制层，视图层；
  - 伪造后端数据，json。已经存在了，不需要后端，前端工程依旧能够跑起来。
- 前后端通过API交互
- 前后端相对独立，松耦合；
- 前后端甚至可以部署在不同的服务器上



前后端分离产生问题：

- 前后端集成联调，无法协调需求与实现，应尽早解决，避免问题集中爆发；



解决方案：

- 首先制定一个Scheme【计划的提纲】，实时更新最新API，降低集成的风险；
- 早些年：制定Word计划文档；
- 前后端分离：
  - 前端测试后端接口：postman
  - 后端提供接口，需要实时更新最新的消息及改动；



## Swagger应运而生

[Swagger官网](https://swagger.io/)

- 号称世界上最流行的API框架；
- RestFul API文档在线自动生成工具=>API文档与API定义同步更新
- 直接运行可以在线测试API接口
- 支持多种语言（java、php......）



在项目中使用Swagger需要导包springfox：

- swagger2
- ui



## SpringBoot集成Swagger

1. 新建一个SpringBoot的Web工程

2. 导入相关依赖：

   - swagger2
   - swagger-ui

   ```xml
   <!-- swagger -->
   <dependency>
       <groupId>io.springfox</groupId>
       <artifactId>springfox-swagger2</artifactId>
       <version>2.9.2</version>
   </dependency>
   <dependency>
       <groupId>io.springfox</groupId>
       <artifactId>springfox-swagger-ui</artifactId>
       <version>2.9.2</version>
   </dependency>
   ```

3. 编写一个HelloWorld

   ```java
   @RestController
   public class HelloController {
       @GetMapping("/hello")
       public String hello(){
           return "hello";
       }
   }
   ```

   访问；

4. 配置Swagger==>config.SwaggerConfig

   ```java
   @Configuration
   @EnableSwagger2 //开启Swagger2
   public class SwaggerConfig {
   }
   ```

5. 测试运行；

   - 重启项目
   - 访问 http://localhost:8080/swagger-ui.html



## 配置Swagger

Swagger的bean实例Docket；

```java
@Configuration
@EnableSwagger2 //开启Swagger2
public class SwaggerConfig {

    /**
     * 配置Swagger的DocketBean实例
     * @return
     */
    @Bean
    public Docket docket(){
        // 文档信息
        DocumentationType documentationType = new DocumentationType("测试Swagger", "1.0版本");
        Docket docket = new Docket(documentationType);
        // 作者信息
        Contact tomxwd = new Contact("tomxwd", "http://blog.tomxwd.top", "648913416@qq.com");
        // 额外信息
        VendorExtension vendorExtension = new VendorExtension() {
            @Override
            public String getName() {
                return "tomxwd";
            }

            @Override
            public Object getValue() {
                return "123";
            }
        };
        // Api信息
        ApiInfo apiInfo = new ApiInfo("测试swagger",
                "第一次使用Swagger建立的项目",
                "1.0",
                "termsOfServiceUrl",
                tomxwd,
                "没有license",
                "没有lincenseUrl", Arrays.asList(vendorExtension));
        docket.apiInfo(apiInfo);
        docket.groupName("swagger-test-group");
        return docket;
    }

}
```

最简单的话，就只用new Docket(DocumentationType)默认的即可；



## Swagger配置扫描接口

Docket.select()

.api(RequestHandlerSelectors.basePackage("top.tomxwd"))

.paths()

.build();

- api()

  - RequestHandlerSelectors配置要扫描接口的方式:
    - basePackage：指定要扫描的包
    - any：扫描全部
    - none：什么都不扫描
    - withMethodAnnotation：通过方法的注解进行扫描
    - withClassAnnotation：扫描类上的注解

  一般用basePackage；

- paths()

  - PathSelectors配置要过滤的路径
    - ant：路径，比如只扫描/hello/**
    - any：所有都过滤
    - none：所有都不过滤
    - regex：正则

  一般用ant

- build()：工厂设计模式，需要最后调用；

```java
@Configuration
@EnableSwagger2 //开启Swagger2
public class SwaggerConfig {

    /**
     * 配置Swagger的DocketBean实例
     * @return
     */
    @Bean
    public Docket docket(){
        // 文档信息
        DocumentationType documentationType = new DocumentationType("测试Swagger", "1.0版本");
        Docket docket = new Docket(documentationType);
        // 作者信息
        Contact tomxwd = new Contact("tomxwd", "http://blog.tomxwd.top", "648913416@qq.com");
        // 额外信息
        VendorExtension vendorExtension = new VendorExtension() {
            @Override
            public String getName() {
                return "tomxwd";
            }

            @Override
            public Object getValue() {
                return "123";
            }
        };
        // Api信息
        ApiInfo apiInfo = new ApiInfo("测试swagger",
                "第一次使用Swagger建立的项目",
                "1.0",
                "termsOfServiceUrl",
                tomxwd,
                "没有license",
                "没有lincenseUrl", Arrays.asList(vendorExtension));
        docket.apiInfo(apiInfo);
        docket.groupName("swagger-test-group");
        // RequestHandlerSelectors配置要扫描接口的方式
        docket.select()
                .apis(RequestHandlerSelectors.basePackage("top.tomxwd.swaggertest.controller"))
                .paths(PathSelectors.ant("/hello/**"))
                .build();
        docket.enable(true);
        return docket;
    }

}
```





## 配置是否启动Swagger

docket.enable(boolean)

true或false开启或关闭；



**如果只希望Swagger在开发环境中使用，在生产的时候不使用：**

- 判断是否生产环境
- 注入enable()

```java
@Bean
// 注入环境对象
public Docket docket(Environment environment){
    // ...省略前面部分代码
    
    // 判断当前环境是否为dev
    Profiles profiles = Profiles.of("dev");
    boolean b = environment.acceptsProfiles(profiles);
    if(b){
        docket.enable(true);
    }
    
    // ...省略后面部分代码
}
```



## 配置API文档的分组

Docket.groupName("group-name")



### 配置多个分组：

配置多个DocketBean实例即可：

```java
@Bean
public Docket docket1(){
    return new Docket(DocumentationType.SWAGGER_2).groupName("A");
}
@Bean
public Docket docket2(){
    return new Docket(DocumentationType.SWAGGER_2).groupName("B");
}
@Bean
public Docket docket3(){
    return new Docket(DocumentationType.SWAGGER_2).groupName("C");
}
```

具体的扫描配置也可以自行定义；



### 实体类配置

1. 只要我们的接口中，返回值中存在实体类，他就会被扫描到Swagger中

   ```java
   // 只要我们的接口中，返回值中存在实体类，他就会被扫描到Swagger中
   @PostMapping("/user")
   public User user(){
       return new User();
   }
   ```

   这时候User就会被扫描到实体类中；

2. 在实体类上打@ApiModel注解，属性上打@ApiModelProperty注解：

   ```java
   @ApiModel("用户实体类")
   @Data
   public class User {
   
       @ApiModelProperty("id")
       private Integer id;
   
       @ApiModelProperty("用户名")
       private String name;
   
       @ApiModelProperty("用户年龄")
       private Integer age;
   
   }
   ```



## 接口信息配置

- @ApiOperation注解
  - 加在方法上，可以描述方法作用
- @ApiParam注解
  - 加载入参上，可以描述入参

