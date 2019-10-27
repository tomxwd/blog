---
title: Maven
date: 2019-09-15 12:14:26
tags: 
 - Java
 - Maven
categories:
 - Maven
---

# Maven

## 下载

[官网下载](http://maven.apache.org/download.cgi)

选择apache-maven-3.6.2-bin.zip即可



## 环境变量配置

- M2_HOME:D:\open\apache-maven-3.5.3
- Path:%M2_HOME%/bin



## 测试

cmd中输入mvn -v出版本信息即可。



## 修改配置

最好复制一份%M2_HOME%/conf/settings.xml到User/.m2/settings.xml。

### 修改JDK默认版本

```xml
<profile>     
    <id>JDK-1.8</id>       
    <activation>       
        <activeByDefault>true</activeByDefault>       
        <jdk>1.8</jdk>       
    </activation>       
    <properties>       
        <maven.compiler.source>1.8</maven.compiler.source>       
        <maven.compiler.target>1.8</maven.compiler.target>       
        <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>       
    </properties>       
</profile>
```

### 添加国内镜像源

```xml
<!-- 阿里云仓库 -->
<mirror>
    <id>alimaven</id>
    <mirrorOf>central</mirrorOf>
    <name>aliyun maven</name>
    <url>http://maven.aliyun.com/nexus/content/repositories/central/</url>
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
```

