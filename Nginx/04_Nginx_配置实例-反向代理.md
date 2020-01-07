---
title: 04_Nginx_配置实例-反向代理
date: 2020-01-06 22:41:11
tags: 
 - Nginx
categories:
 - Nginx
---

# 04_Nginx_配置实例-反向代理

## 反向代理实例1

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
     192.168.2.128	www.123.com
     ```

3. 访问过程分析

   通过访问123.com，直接访问到tomcat里面， 注意这里Nginx开放的是80端口，而tomcat开放的是8080端口，所以我们要做的是屏蔽掉tomcat对外暴露的端口，仅通过Nginx的80端口访问Tomcat；
   
4. 修改Nginx配置文件nginx.conf

   - 修改默认的server块中，server_name由localhost改为虚拟机ip
   - server块中，location加到root下面，添加`proxy_pass http://127.0.0.1:8080;`

   ```shell
   server {
           listen       80;
           server_name  192.168.2.128;
   
           #charset koi8-r;
   
           #access_log  logs/host.access.log  main;
   
           location / {
               root   html;
               proxy_pass   http://127.0.0.1:8080;
               index  index.html index.htm;
           }
   }
   ```

5. 启动Nginx

6. 访问www.123.com应该要跳转到tomcat页面，同时，8080端口其实可以不对外开放



## 反向代理实例2

1. 实现效果：

   使用nginx反向代理，根据访问的路径跳转到不同的端口服务中

   nginx监听9001端口

   访问http://www.123.com:9001/edu/ 直接跳转到127.0.0.1:8080

   访问http://www.123.com:9001/vod/ 直接跳转到127.0.0.1:8081

2. 准备工作

   - 准备两个tomcat服务器，一个8080，一个8081

   - 8081tomcat需要配置端口号，在/conf/server.xml中配置

     `<Server port="8005" shutdown="SHUTDOWN">`

     改为：

     `<Server port="8015" shutdown="SHUTDOWN">`，

     再把

     ```shell
     <Connector port="8080" protocol="HTTP/1.1"
                connectionTimeout="20000"
                redirectPort="8443" />
     ```
     改为：

     ```shell
     <Connector port="8081" protocol="HTTP/1.1"
                connectionTimeout="20000"
                redirectPort="8443" />
     ```

     再把

     ```shell
     <Connector port="8009" protocol="AJP/1.3" redirectPort="8443" />
     ```

     改为：

     ```shell
     <Connector port="8019" protocol="AJP/1.3" redirectPort="8443" />
     ```

   - 创建文件夹和测试的页面

     在8080端口的tomcat中的webapps下创建edu文件夹，编写a.html

     ```html
     <h1>
         8080!!!!!!!!!
     </h1>
     ```

     在8081端口的tomcat中的webapps下创建vod文件夹，编写a.html

     ```html
     <h1>
         8081!!!!!!!!!
     </h1>
     ```
     
   - 如果没有开放9001端口：

     ` firewall-cmd --add-port=9001/tcp --permanent`

     `firewall-cmd --reload`

3. Nginx具体配置

   添加一段server配置

   ```shell
   server{
   	listen  9001;
   	server_name 192.168.2.128;
   
   	location ~ /edu/ {
   	proxy_pass http://localhost:8080;
   	}
   
   	location ~ /vod/ {
   	proxy_pass http://localhost:8081;
   	}
   }
   ```

4. 测试：

   - 访问http://www.123.com:9001/edu/a.html
   - 访问http://www.123.com:9001/vod/a.html
   - 查看结果





