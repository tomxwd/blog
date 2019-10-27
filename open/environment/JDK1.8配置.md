---
title: JDK1.8配置
date: 2019-09-13 18:57:37
tags: 
 - Java
 - JDK
categories:
 - JDK
---

# JDK1.8配置

环境变量：

- JAVA_HOME:D:\open\jdk1.8.0_45

- Path:%JAVA_HOME%\bin;%JAVA_HOME%\jre\bin;
- CLASSPATH:.;%JAVA_HOME%\lib;%JAVA_HOME%\lib\tools.jar**【注意前面，配个点】**

如果是多jdk版本的话，直接配多几个类似JAVA7_HOME、JAVA8_HOME，然后修改JAVA_HOME的值为指定版本的home即可。

输入java -version检查是否输出jdk版本，有即可。

