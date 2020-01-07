---
title: 06_Nginx_配置实例-负载均衡
date: 2020-01-07 22:20:47
tags: 
 - Nginx
categories:
 - Nginx
---

# 06_Nginx_配置实例-负载均衡

1. 实现效果

   - 访问http://www.123.com/edu/a.html，达到负载均衡的效果，也就是把请求平均分发到8081以及8082中；

2. 准备工作

   两台tomcat服务器，一台8080，一台8081

   在两台tomcat的webapps中，都创建edu文件夹，以及a.html，内容分别为`<h1>8080!!!</h1>`以及`<h1>8081</h1>`

3. 配置nginx.conf

   ```shell
   http{
   ......
   	upstream myserver{
   		server 192.168.2.128:8080;
   		server 192.168.2.128:8081;
   	}
   ......
   	server{
   		location / {
   			......
   			proxy_pass http://myserver;
   		}
   		......
   	}
   }
   ```

4. 测试

   访问http://www.123.com/edu/a.html，刷新几次，查看页面变化

   可以看到8080和8081交替被访问的效果



## 负载均衡

随着互联网信息的爆炸性增长，负载均衡（load balance）已经不再是一个很陌生的话题，顾名思义，负载均衡即是将负载分摊到不同的服务单元，既保证服务的可用性，又保证响应足够快，给用户很好的体验。快速增长的访问量和数据流量催生了各式各样的负载均衡产品，很多专业的负载均衡硬件提供了很好的功能，但是价格不菲，这使得负载均衡软件大受欢迎，Nginx就是其中一个。在Linux中有Nginx、LVS、Haproxy等等服务可以提供负载均衡服务，而Nginx提供了以下几种分配方式（策略）：

1. 轮询（默认）

   每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器down掉，能自动剔除。

2. weight（权重）

   **在upstream块中的server ip:prot 后面添加weight=xxx；**

   weight代表权重，默认是1，权重越高分配到的客户端越多；

   指定轮询几率，weight和访问比率成正比，用于后端服务器性能不均的情况，例如

   ```shell
   upstream myserver{
   	server 192.168.2.128:8080 weight=10;
   	server 192.168.2.128:8081 weight=10;
   }
   ```

3. ip_hash

   **在upstream块中添加ip_hash；**

   每个请求按访问ip的hash结果分配，这样每个访客固定访问一个后端服务器，**可以解决session问题**，例如

   ```shell
   upstream myserver{
   	ip_hash;
   	server 192.168.2.128:8080;
   	server 192.168.2.128:8081;
   }
   ```

4. fair（第三方）

   按后端服务器的**响应时间来分配请求**，响应时间短的优先分配。例如：

   ```shell
   upstream myserver{
   	fair;
   	server 192.168.2.128:8080;
   	server 192.168.2.128:8081;
   }
   ```







