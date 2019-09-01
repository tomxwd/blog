---
title: 05_SpringBoot_Web开发
date: 2019-07-08 20:15:38
tags: 
 - Java
 - SpringBoot
categories:
 - Spring
 - SpringBoot初学
---

# 05_SpringBoot_Web开发

使用SpringBoot；

1. **创建SpringBoot应用，选中我们需要的模块；**
2. **SpringBoot已经默认将这些场景配置好了，只需要在配置文件中指定少量配置就可以运行起来**
3. **自己编写业务代码；**



## 自动配置原理

这个场景SpringBoot帮我们配置了什么？能不能修改？能修改哪些配置？能不能扩展？。。。

**`XxxAutoConfiguration`：帮我们给容器中自动配置组件**

**`XxxProperties`：配置类来封装配置文件的内容；**



## SpringBoot对静态资源的映射规则

```java
@ConfigurationProperties(prefix = "spring.resources", ignoreUnknownFields = false)
public class ResourceProperties {
```

**可以设置和静态资源有关的参数，缓存时间等。**

```java
@Override
public void addResourceHandlers(ResourceHandlerRegistry registry) {
    if (!this.resourceProperties.isAddMappings()) {
        logger.debug("Default resource handling disabled");
        return;
    }
    Duration cachePeriod = this.resourceProperties.getCache().getPeriod();
    CacheControl cacheControl = this.resourceProperties.getCache().getCachecontrol().toHttpCacheControl();
    if (!registry.hasMappingForPattern("/webjars/**")) {
        customizeResourceHandlerRegistration(registry.addResourceHandler("/webjars/**")
.addResourceLocations("classpath:/META-INF/resources/webjars/") .setCachePeriod(getSeconds(cachePeriod)).setCacheControl(cacheControl));
        
    }
    String staticPathPattern = this.mvcProperties.getStaticPathPattern();
    if (!registry.hasMappingForPattern(staticPathPattern)) {
        customizeResourceHandlerRegistration(registry.addResourceHandler(staticPathPattern)
                                             .addResourceLocations(getResourceLocations(this.resourceProperties.getStaticLocations()))
                                             .setCachePeriod(getSeconds(cachePeriod)).setCacheControl(cacheControl));
    }
}
```

1. 所有/webjars/**，都去classpath:/META-INF/resources/webjars/找资源；

   - webjars：以jar包的方式引入静态资源；

   - [webjars官网](https://www.webjars.org/)

   - ![引入jqueryPOM依赖](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/SpringBoot/05%E5%BC%95%E5%85%A5jqueryPOM%E4%BE%9D%E8%B5%96.png)

   - localhost:8080/webjars/jquery/3.3.1/jquery.js

   - ```xml
     <!-- webjar的形式引入jquery3.3.1 -->
     <dependency>
         <groupId>org.webjars</groupId>
         <artifactId>jquery</artifactId>
         <version>3.3.1</version>
     </dependency>
     ```

   - 在访问的时候只需要写webjars下面资源的名称即可

2. `private String staticPathPattern = "/**"`访问当前项目的任何资源，（静态资源的文件夹）

   ```java
   private static final String[] CLASSPATH_RESOURCE_LOCATIONS = { 
   "classpath:/META-INF/resources/",
   "classpath:/resources/",                                       "classpath:/static/",                                           "classpath:/public/" 
   };
   ```

   老版本是还有一个`"/"`当前项目下的根路径。

   localhost:8080/abc======>去静态资源文件夹里面找abc



```xml
 <!-- 阿里云仓库 -->
        <mirror>
            <id>alimaven</id>
           <mirrorOf>central</mirrorOf>
            <name>aliyun maven</name>
            <url>https://maven.aliyun.com/nexus/content/repositories/central/</url>
         </mirror>
     
        <!-- 中央仓库1 -->
        <mirror>
            <id>repo1</id>
             <mirrorOf>central</mirrorOf>
            <name>Human Readable Name for this Mirror.</name>
            <url>http://repo1.maven.org/maven2/</url>
         </mirror>
     
         <!-- 中央仓库2 -->
         <mirror>
             <id>repo2</id>
             <mirrorOf>central</mirrorOf>
             <name>Human Readable Name for this Mirror.</name>
             <url>http://repo2.maven.org/maven2/</url>
         </mirror>

<repositories>
    <repository>
        <id>central</id>
        <url>http://host:port/content/groups/public</url>
    </repository>
</repositories>
 
<pluginRepositories>
    <pluginRepository>
        <id>central</id>
        <url>http://host:port/content/groups/public</url>
    </pluginRepository>
</pluginRepositories>
```

