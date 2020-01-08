---
title: 07_Nginx_配置实例-动静分离
date: 2020-01-08 20:48:25
tags: 
 - Nginx
categories:
 - Nginx
---

# 07_Nginx_配置实例-动静分离

Nginx动静分离简单来说就是把**动态跟静态请求**分开，不能理解成只是单纯把动态页面和静态页面物理分离，严格意义上说应该是动态请求跟静态请求分开，可以理解成使用Nginx处理静态页面，Tomcat处理动态页面。动静分离从目前实现角度来讲大致分为两种：

1. 一种是纯粹把静态文件独立成单独的域名，放在独立的服务器上，也是目前主流推崇的方案；
2. 另一种方法就是动态跟静态文件混合在一起发布，通过Nginx分开。

![image-20200108210311550](07_Nginx_%E9%85%8D%E7%BD%AE%E5%AE%9E%E4%BE%8B-%E5%8A%A8%E9%9D%99%E5%88%86%E7%A6%BB/image-20200108210311550.png)

**通过location指定不同的后缀名实现不同的请求转发。**

**通过expires参数，可以设置浏览器缓存过期时间**，减少与服务器之间的请求和流量。

**具体Expires定义：**给资源设定过期时间，也就是说无需去服务端验证，直接通过浏览器自身确认是否过期即可，所以不会产生额外的流量。这种方式非常适合不经常变动的资源。（如果经常更新的文件，不建议使用Expires来缓存），这里设置3d，代表3天内访问这个URL，发送同一个请求，比对服务器该文件最后更新时间没有变化，则不会从服务器抓取，返回状态码304，如果有修改，则直接从服务器重新下载，返回状态码200.



## 动静分离

1. 项目资源准备

   在Linux系统中准备静态资源，用于进行访问

   在/home/目录下创建data/image文件夹以及data/www文件夹，在image中放入01.png图片，在www中放入a.html静态页面；

2. 进行Nginx配置

   到/usr/local/nginx/conf，编辑nginx.conf文件

   ```shell
   server {
   	listen       80;
   	server_name  192.168.2.128;
   
   	#charset koi8-r;
   
   	#access_log  logs/host.access.log  main;
   
   	location /www/ {
   		root   /home/data/;
   		#            proxy_pass   http://myserver;
   		index  index.html index.htm;
   	}
   
   	location /image/ {
   		root /home/data/;
   		autoindex on;
   	}
   }
   ```

   修改location块，修改root为/data/，添加一个location块，image配置中加入autoindex on，可以开启index索引效果；

3. 测试：

   访问http://www.123.com/image/和http://www.123.com/image/01.png以及http://www.123.com/www/a.html

   注意这里访问image需要加/，否则找不到index页面







