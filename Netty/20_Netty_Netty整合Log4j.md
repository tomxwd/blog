---
title: 20_Netty_Netty整合Log4j
date: 2019-12-26 21:11:37
tags: 
 - Netty
categories:
 - Netty
---

# 20_Netty_Netty整合Log4j

1. maven添加log4j依赖

   ```xml
   <dependency>
       <groupId>log4j</groupId>
       <artifactId>log4j</artifactId>
       <version>1.2.17</version>
   </dependency>
   <dependency>
       <groupId>org.slf4j</groupId>
       <artifactId>slf4j-api</artifactId>
       <version>1.7.29</version>
   </dependency>
   <dependency>
       <groupId>org.slf4j</groupId>
       <artifactId>slf4j-log4j12</artifactId>
       <version>1.7.29</version>
   </dependency>
   <dependency>
       <groupId>org.slf4j</groupId>
       <artifactId>slf4j-simple</artifactId>
       <version>1.7.29</version>
       <scope>test</scope>
   </dependency>
   ```

2. 配置log4j，在resources/log4j.properties

   ```properties
   log4j.rootLogger=DEBUG, stdout
   log4j.appender.stdout=org.apache.log4j.ConsoleAppender
   log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
   log4j.appender.stdout.layout.ConversionPattern=[%p] %C{1} - %m%n
   ```

