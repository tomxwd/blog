---
title: Maven导不进包的解决办法
date: 2019-07-08 21:48:50
tags: 
 - Java
 - Maven
categories:
 - Java环境
 - Maven
---

# Maven导不进包的解决办法

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

在项目的pom文件中导入
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

