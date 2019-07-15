---
title: 04_SpringBoot_日志
data: 2019-07-07 ‏‎17:43:52
tags: 
 - Java
 - SpringBoot
categories:
 - Spring
 - SpringBoot初学
---

# 04_SpringBoot_日志

## 1.日志框架

#### 日志框架小故事

小张：开发一个大型系统：

1. System.out.println("");将关键数据打印在控制台；老板说要去掉，全部注释了，结果老板说好像输出数据确实有需要，可以写在一个文件？监控一下系统运作？
2. 框架来记录系统的一些信息：日志框架：zhanglogging.jar；
3. 高大上的几个功能？异步模式？自动归档？XXX?zhanglogging-good.jar?
4. 将以前框架卸下来？换上新的框架，重新修改之前相关的API；zhanglogging-perfect.jar;
5. 类似JDBC-----数据库驱动；
   - 写了一个统一的接口层；日志门面（日志的一个抽象层）；logging-abstract.jar
   - 给项目中导入具体的日志实现就行了;我们之前的日志框架都是实现了抽象层；



#### 市面上的日志框架

JUL、JCL、Jboss-logging、logback、log4j、log4j2、slf4j......

| 日志门面                                                     | 日志实现                                                     |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 1.**~~JCL(Jakarta Commons Logging)~~**<br />2.SLF4j(Simple Logging Facade for java)<br />3.**~~jboss-logging~~** | 1.log4j<br />2.JUL(java.util.logging  java自带的)<br />3.Log4j2<br />4.**Logback** |

左边选一个门面（抽象层），右边选一个实现；

日志门面：SLF4J；（SLF4j，log4j，Logback出自同一个人）

日志实现：Logback



SpringBoot：底层是Spring框架，Spring框架默认是使用JCL；

- **SpringBoot在starter-logging选用SLF4j和logback；**
- 内容上跟Spring一样使用JCL
- SpringBoot也可以自动适配（JUL、log4j2、logback）并简化配置



## 2.SLF4j使用

#### 如何在系统中使用SLF4j？（使用原理）

> [SLF4j官网](https://www.slf4j.org/)

以后开发的时候，日志记录方法的调用，不应该来直接调用日志的实现类，而是调用日志抽象层里的方法；

在官网主页点击[SLF4J user manual](https://www.slf4j.org/manual.html)

给系统里面倒入slf4j的jar和logback的实现jar

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class HelloWorld {
  public static void main(String[] args) {
    Logger logger = LoggerFactory.getLogger(HelloWorld.class);
    logger.info("Hello World");
  }
}
```



![SLF4j抽象与实现](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/SpringBoot/04SLF4j%E6%8A%BD%E8%B1%A1%E4%B8%8E%E5%AE%9E%E7%8E%B0.png)

每一个日志的实现框架都有自己的配置文件。

使用slf4j以后，**配置文件还是做成日志实现框架自己本身的配置文件**



#### 其他日志框架统一转换为slf4j

遗留问题：

a(slf4j+logback):Spring（commons-logging)、Hibernate(jboss-logging)、Mybatis、.....XXX

统一日志记录，即使是别的框架，也要和我一起统一使用slf4j进行输出

查看官网下的[legacy APIs](https://www.slf4j.org/legacy.html)

![统一转换slf4j](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/SpringBoot/04%E7%BB%9F%E4%B8%80%E8%BD%AC%E6%8D%A2slf4j.png)

**如何让系统中所有的日志都统一到slf4j**

1. **将系统中其他日志框架先排除出去；（这一步会导致框架启动不起来，因为排除了日志包。）**
2. **用中间包来替换原有的日志框架；（这一步框架不会报错了）**
3. **我们导入slf4j其他的实现**

SpringBoot也是这么来处理的。



## 3.SpringBoot日志关系

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
    <version>2.1.6.RELEASE</version>
    <scope>compile</scope>
</dependency>
```

SpringBoot使用它来做日志功能：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-logging</artifactId>
    <version>2.1.6.RELEASE</version>
    <scope>compile</scope>
</dependency>
```

底层依赖关系

![老版本logging-start](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/SpringBoot/04%E8%80%81%E7%89%88%E6%9C%AClogging-start.png)

上面是老版本的

新版本无JCL。

**总结：**

1. SpringBoot底层也是使用slf4j+logback的方式进行日志记录

2. SpringBoot也把其他的日志都替换成了slf4j；

3. 中间替换包？

   ![偷梁换柱](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/SpringBoot/04%E5%81%B7%E6%A2%81%E6%8D%A2%E6%9F%B1.png)

4. 如果我们要引入其他框架？一定要把这个框架的默认日志依赖移除掉！！！

   - Spring框架用的是commons-logging

     ![SpringBoot排除日志依赖](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/SpringBoot/04SpringBoot%E6%8E%92%E9%99%A4%E6%97%A5%E5%BF%97%E4%BE%9D%E8%B5%96.png)

   - **SpringBoot能自动适配所有的日志，而且底层使用slf4j+logback的方式记录日志，引入其他框架的时候，只需要把这个框架依赖的日志框架排除掉即可。**



## 4.日志使用

> [可以在spring官网看关于logging的配置相关介绍，有专门的章节做讲解。](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-logging.html)

#### 默认配置

**级别**

SpringBoot默认级别：root

SpringBoot默认帮我们配置好了日志

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringBoot03LoggingApplicationTests {

    // 记录器
    Logger logger = LoggerFactory.getLogger(getClass());

    @Test
    public void contextLoads() {

        // 日志的级别
        // 由低到高 trace<debug<info<warn<error
        // 可以调整输出的日志级别；日志就只会在这个级别以及以后的高级别生效
        logger.trace("这是trace日志。。。");
        logger.debug("这是debug日志。。。");
        // SpringBoot默认使用info级别的,可以在application.properties配置文件里调整级别
        // 没有指定级别的就使用SpringBoot默认级别；root级别
        logger.info("这是info日志。。。");
        logger.warn("这是warn日志。。。");
        logger.error("这是error日志。。。");


    }

}
```

日志级别：低到高 trace<debug<info<warn<error

SpringBoot默认是info，可以在application.properties配置文件调整级别。



#### 指定配置

二选一：

logging.file和logging.path

都指定，就logging.file起作用

| logging.file | logging.path | Example  | Description                      |
| ------------ | ------------ | -------- | -------------------------------- |
| (none)       | (none)       |          | 只在控制台输出                   |
| 指定文件名   | (none)       | my.log   | 输出日志到my.log文件             |
| (none)       | 指定目录     | /var/log | 输出到指定目录的spring.log文件中 |

```properties
logging.level.com.github.tomxwd=trace



# 不指定路径当前项目下生成springboot.log日志
logging.file=/springboot.log
# 可以指定完整的路径：
# logging.file=F:/springboot.log

# 在当前磁盘的根路径(项目路径所在的磁盘根路径)下创建spring文件夹和里面的log文件夹；使用spring.log作为默认文件
# logging.path=/spring/log

# 日志输出格式
# %d表示日期时间
# %thread表示线程名
# %-5level：级别从左显示5个字符宽度
# %logger[50] 表示logger名字最长50个字符，否则按照句点分割
# %msg： 日志消息
# %n是换行符

# 在控制台输出的日志格式
logging.pattern.console=%d{yyyy-MM-dd} [%thread] %-5level %logger{50} - %msg%n

# 指定文件中日志输出的格式
logging.pattern.file=%d{yyyy-MM-dd HH:mm:ss.SSS} === [%thread] === %-5level === %logger{50} === - %msg%n
```

- 日志输出格式
  - %d表示日期时间
  - %thread表示线程名
  - %-5level：级别从左显示5个字符宽度
  - %logger[50] 表示logger名字最长50个字符，否则按照句点分割
  - %msg： 日志消息
  - %n是换行符
- 示例
  - `%d{yyyy-MM-dd} [%thread] %-5level %logger{50} - %msg%n`



给类路径下放上每个日志框架自己的配置文件即可；SpringBoot就会不使用他默认配置的了

![指定配置文件.png](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/SpringBoot/04%E6%8C%87%E5%AE%9A%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6.png)

| Logging System          | Customization                                                |
| ----------------------- | ------------------------------------------------------------ |
| Logback                 | `logback-spring.xml`, `logback-spring.groovy`, `logback.xml`, or `logback.groovy` |
| Log4j2                  | `log4j2-spring.xml` or `log4j2.xml`                          |
| JDK (Java Util Logging) | `logging.properties`                                         |

logback.xml：直接就被日志框架识别了；

**logback-spring.xml**：加上-spring就不会直接加载日志的配置项，就交给SpringBoot来加载，高级特性，SpringBoot可以用来做profile指定。

```xml
<springProfile name="staging">
	<!-- configuration to be enabled when the "staging" profile is active -->
</springProfile>

<springProfile name="dev | staging">
	<!-- configuration to be enabled when the "dev" or "staging" profiles are active -->
</springProfile>

<springProfile name="!production">
	<!-- configuration to be enabled when the "production" profile is not active -->
</springProfile>
```

**可以指定某段配置只在某个环境下生效！**

例如：

```xml
<appender name="stdout" class="ch.qos.logback.core.ConsoleAppender">
 <!--
   日志输出格式：
   %d表示日期时间，
   %thread表示线程名，
   %-5level：级别从左显示5个字符宽度
   %logger{50} 表示logger名字最长50个字符，否则按照句点分割。 
   %msg：日志消息，
   %n是换行符
        -->
    <layout class="ch.qos.logback.classic.PatternLayout">
        <springProfile name="dev">
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} ----> [%thread] ---> %-5level %logger{50} - %msg%n</pattern>
        </springProfile>
        <springProfile name="!dev">
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} ==== [%thread] ==== %-5level %logger{50} - %msg%n</pattern>
        </springProfile>
    </layout>
</appender>
```

如果想要指定配置环境又不使用-spring后缀，那么会报错。



#### 切换日志框架

> 首选还是用[官网的手段](https://docs.spring.io/spring-boot/docs/current/reference/html/using-boot-build-systems.html#using-boot-starter)
>
> 把默认的spring-boot-starter-logging用spring-boot-starter-log4j2来替换就好了
>
> 以下只是为了演示原理的东西

**演示版本**

例如slf4j--->log4j

1. 首先排除logback-classic

2. 要排除log4j转slf4j的jar（log4j-over-slf4j)（高版本为log4j-to-slf4j）

   ```xml
   <dependencies>
       <dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-web</artifactId>
           <exclusions>
               <exclusion>
                   <artifactId>logback-classic</artifactId>
                   <groupId>ch.qos.logback</groupId>
               </exclusion>
               <exclusion>
                   <artifactId>log4j-to-slf4j</artifactId>
                   <groupId>org.apache.logging.log4j</groupId>
               </exclusion>
           </exclusions>
       </dependency>
   
       <dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-test</artifactId>
           <scope>test</scope>
       </dependency>
   
       <dependency>
           <groupId>org.slf4j</groupId>
           <artifactId>slf4j-log4j12</artifactId>
       </dependency>
   
   </dependencies>
   ```

3. 导入slf4j转log4j的包，这个包已经包含了log4j的实现依赖，所以下一步不需要到log4j实现包。

   ```xml
   <dependency>
       <groupId>org.slf4j</groupId>
       <artifactId>slf4j-log4j12</artifactId>
   </dependency>
   ```

   这时候需要导入log4j.properties配置文件

   ```properties
   # Configure logging for testing: optionally with log file
   log4j.rootLogger=INFO, stdout
   # log4j.rootLogger=WARN, stdout, logfile
   
   log4j.appender.stdout=org.apache.log4j.ConsoleAppender
   log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
   log4j.appender.stdout.layout.ConversionPattern=%d %p [%c] - %m%n
   
   log4j.appender.logfile=org.apache.log4j.FileAppender
   log4j.appender.logfile.File=target/spring.log
   log4j.appender.logfile.layout=org.apache.log4j.PatternLayout
   log4j.appender.logfile.layout.ConversionPattern=%d %p [%c] - %m%n
   ```

   就会发现确实是log4j的配置输出。

我们可以按照slf4j的日志适配图，进行相关的切换；slf4j->log4j的方式



**官网手段**

在[官网starter](https://docs.spring.io/spring-boot/docs/current/reference/html/using-boot-build-systems.html#using-boot-starter)里面，有关于如何切换日志的手段：

1. 首先排除SpringBoot本身使用的spring-boot-starter-logging
2. 导入spring-boot-starter-log4j2即可

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <exclusions>
            <exclusion>
                <artifactId>spring-boot-starter-logging</artifactId>
                <groupId>org.springframework.boot</groupId>
            </exclusion>
        </exclusions>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>

    <!-- log4j starter -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-log4j2</artifactId>
    </dependency>

</dependencies>
```

这里，SpringBoot已经写好了一个默认的log4j配置文件（log4j2.xml）又由于xml优先级大于properties，所以就算你上一步写了log4j.properties文件，也会被覆盖配置，因此你可以写一个log4j2.xml来替换他的默认配置，当然最好还是写log4j2-spring.xml，可以使用SpringBoot的增强功能，支持为环境做不同的配置。

