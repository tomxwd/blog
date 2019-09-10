---
title: 第5章_Maven坐标和依赖
date: 2019-09-05 22:46:03
tags: 
 - Java
 - Maven
categories:
 - Maven
---

# 第5章_Maven坐标和依赖



## account-email

### pom依赖文件

pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>top.tomxwd.mvnbook.account</groupId>
    <artifactId>account-email</artifactId>
    <version>1.0-SNAPSHOT</version>

    <!-- 依赖 -->
    <dependencies>
        <!-- Spring -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
            <version>5.0.5.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-beans</artifactId>
            <version>5.0.5.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.0.5.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context-support</artifactId>
            <version>5.0.5.RELEASE</version>
        </dependency>
        <!-- mail邮件服务 -->
        <dependency>
            <groupId>javax.mail</groupId>
            <artifactId>mail</artifactId>
            <version>1.4.1</version>
        </dependency>
        <!-- 测试 -->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
        <!-- 邮件测试 -->
        <dependency>
            <groupId>com.icegreen</groupId>
            <artifactId>greenmail</artifactId>
            <version>1.3.1b</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <!-- 插件 -->
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>
```

greenmail是开源的邮件服务测试套件，测试邮件的发送。详情可以看[GreenMail官网](http://www.icegreen.com/greenmail/)介绍；

### 主代码

AccountEmailService接口：

```java
package top.tomxwd.mvnbook.account.email;

public interface AccountEmailService {
    void sendMail(String to,String subject,String htmlText);
}
```

AccountEmailServiceImpl实现类：

```java
package top.tomxwd.mvnbook.account.email;

import org.springframework.mail.javamail.JavaMailSender;
import org.springframework.mail.javamail.MimeMessageHelper;

import javax.mail.MessagingException;
import javax.mail.internet.MimeMessage;

public class AccountEmailServiceImpl implements AccountEmailService {

    private JavaMailSender javaMailSender;

    private String systemEmail;

    @Override
    public void sendMail(String to, String subject, String htmlText) {
        try {
            MimeMessage msg = javaMailSender.createMimeMessage();
            MimeMessageHelper msgHelper = new MimeMessageHelper(msg);

            msgHelper.setFrom(systemEmail);
            msgHelper.setTo(to);
            msgHelper.setSubject(subject);
            msgHelper.setText(htmlText,true);

            javaMailSender.send(msg);
        } catch (MessagingException e) {
            e.printStackTrace();
        }
    }

    public JavaMailSender getJavaMailSender() {
        return javaMailSender;
    }

    public void setJavaMailSender(JavaMailSender javaMailSender) {
        this.javaMailSender = javaMailSender;
    }

    public String getSystemEmail() {
        return systemEmail;
    }

    public void setSystemEmail(String systemEmail) {
        this.systemEmail = systemEmail;
    }
}
```

也就是一个简单的，发送一封邮件，来自（From），送往（To），主题（Subject），内容（Text），指定是否为html格式（true），发送（send）即可。加上getter、setter方法用于spring注入用。

account-email.xml（邮件相关配置）

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- 读取配置文件 -->
    <bean id="propertyConfigurer" class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
        <property name="location" value="classpath:service.properties"/>
    </bean>

    <bean id="javaMailSender" class="org.springframework.mail.javamail.JavaMailSenderImpl">
        <property name="protocol" value="${email.protocol}"/>
        <property name="host" value="${email.host}"/>
        <property name="port" value="${email.port}"/>
        <property name="username" value="${email.username}"/>
        <property name="password" value="${email.password}"/>
        <property name="javaMailProperties">
            <props>
                <prop key="mail.${email.protocol}.auth">${email.auth}</prop>
            </props>
        </property>
    </bean>

    <bean id="accountEmailService" class="top.tomxwd.mvnbook.account.email.AccountEmailServiceImpl">
        <property name="javaMailSender" ref="javaMailSender"/>
        <property name="systemEmail" value="${email.systemEmail}"/>
    </bean>

</beans>
```

