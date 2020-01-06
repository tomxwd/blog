---
title: 04_Nginx_配置实例-反向代理
date: 2020-01-06 22:41:11
tags: 
 - Nginx
categories:
 - Nginx
---

# 04_Nginx_配置实例-反向代理

1. 实现效果

   - 打开浏览器，在浏览器地址栏中输入地址www.123.com，跳转到linux系统tomcat主页面中

2. 准备工作

   - 安装jdk

     解压tar包

     etc/profile配置文件追加

     ```shell
     # java 环境配置
     export JAVA_HOME=/opt/open/environment/jdk/jdk1.8.0_221
     export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
     export PATH=$PATH:$JAVA_HOME/bin
     ```

     source /etc/profile让配置生效

   - 安装tomcat

     解压tar包

     运行tomcat：到tomcat/bin目录下./startup.sh即可

     对外开放访问端口：`firewall-cmd --add-port=8080/tcp --permanent`

     重启防火墙：`firewall-cmd --reload`

     查看已经开放的端口号：`firewall-cmd --list-all`

     在windows端直接访问ip:8080看看能不能访问到tomcat

   - 修改windows端对虚拟机的host

     C:\Windows\System32\drivers\etc目录下的hosts文件追加

     ```
     192.168.2.128	123.com
     ```

3. 访问过程分析

   通过访问123.com，直接访问到tomcat里面， 注意这里Nginx开放的是80端口，而tomcat开放的是8080端口，所以我们要做的是屏蔽掉tomcat对外暴露的端口，仅通过Nginx的80端口访问Tomcat；