---
title: 13_Spring集成RabbitMQ-Client
data: 2019-07-01 19:26:40
tags: 
 - Java
 - RabbitMQ
categories:
 - MQ
 - RabbitMQ
---

# 13_Spring集成RabbitMQ-Client

> spring封装了client —— Spring AMQP
>
> Spring AMQP需要实现spring-amqp
>
> 是spring实现的spring-rabbit



---

## 配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:rabbit="http://www.springframework.org/schema/rabbit"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/rabbit http://www.springframework.org/schema/rabbit/spring-rabbit.xsd">
    <!-- 1. 定义RabbitMQ的连接工厂-->
    <rabbit:connection-factory id="connectionFactory" host="127.0.0.1" port="5672" username="user_xwd" password="user_xwd" virtual-host="/vhost_xwd"/>

    <!-- 2. 定义Rabbit模板，指定连接工厂以及定义exchange-->
    <rabbit:template id="rabbitTemplate" connection-factory="connectionFactory" exchange="spring_exchange"></rabbit:template>
    <!-- MQ的管理，包括队列，交换器等的声明 -->
    <rabbit:admin connection-factory="connectionFactory" />

    <!-- 定义队列 自动声明 -->
    <rabbit:queue name="myQueue" auto-declare="true" durable="true"/>

    <!-- 定义交换器，自动声明 -->
    <rabbit:fanout-exchange name="spring_exchange" auto-declare="true">
        <rabbit:bindings>
            <rabbit:binding queue="myQueue"></rabbit:binding>
        </rabbit:bindings>
    </rabbit:fanout-exchange>

    <!-- 队列监听 -->
    <rabbit:listener-container connection-factory="connectionFactory">
        <rabbit:listener ref="consumer" method="listen" queue-names="myQueue"/>
    </rabbit:listener-container>

    <!-- 消费者 -->
    <bean id="consumer" class="com.github.tomxwd.mq.spring.Consumer"/>


</beans>
```



---

## 监听类

```java
package com.github.tomxwd.mq.spring;

/**
 * @author xieweidu
 * @createDate 2019-07-01 20:58
 */
public class Consumer {

    public void listen(String foo){
        // 执行具体的业务方法
        System.out.println("消费者 : "+foo);
    }

}
```



---

## 测试类

```java
/**
 * @author xieweidu
 * @createDate 2019-07-01 20:54
 */
public class SpringMain {

    public static void main(String[] args) throws InterruptedException {
        final AbstractApplicationContext ctx = new ClassPathXmlApplicationContext("classpath:application.xml");
        // RabbitMQ模板
        final RabbitTemplate template = ctx.getBean(RabbitTemplate.class);
        // 发送信息
        template.convertAndSend("Hello World");
        Thread.sleep(1000); // 休眠一秒
        ctx.destroy();  // 销毁容器
    }

}
```



---

## log日志文件

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



---

## 总结

把各种信息配置好，需要用到RabbitMQ的地方引入模板类，然后调用convertAndSend(Object obj)方法即可。

```java
@Autowired
private RabbitTemplate template;
```

