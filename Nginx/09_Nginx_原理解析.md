---
title: 09_Nginx_原理解析
date: 2020-01-09 21:57:35
tags: 
 - Nginx
categories:
 - Nginx
---

# 09_Nginx_原理解析

![image-20200109220043584](09_Nginx_%E5%8E%9F%E7%90%86%E8%A7%A3%E6%9E%90/image-20200109220043584.png)



![image-20200109220122777](09_Nginx_%E5%8E%9F%E7%90%86%E8%A7%A3%E6%9E%90/image-20200109220122777.png)

![image-20200109220204775](09_Nginx_%E5%8E%9F%E7%90%86%E8%A7%A3%E6%9E%90/image-20200109220204775.png)

worker之间是争抢的机制



## master-workers机制的好处

- 首先，对于每个Worker进程来说，独立的进程，不需要加锁，所以省掉了锁带来的开销，同时在编程以及问题查找的时候，也会方便很多。

- 其次，采用独立的进程，可以让进程互相之间不会影响，一个进程退出之后，其他进程还在工作，服务不会中断，master进程则很快启动新的worker进程。

- 当然，worker进程的异常退出，肯定是程序有bug了，异常退出，会导致当前worker上所有的请求失败，不过不会影响所有请求，所以降低了风险；

- 还可以使用nginx -s reload 热部署，因为master-workers的机制，不会影响到已经进来的请求，相当于分批进行热部署。



### 需要设置多少个worker

Nginx同redis类似都采用了io多路复用机制，每个worker都是一个独立的进程，但每个进程只有一个主线程，通过异步非阻塞的方式来处理请求，即使是成千上万个请求也不在话下。每个worker的线程可以把一个cpu的性能发挥到极致。所以worker数和服务器的cpu数相等是最为适宜的，设置少了会浪费CPU，多了会造成CPU频繁切换上下文带来损耗。



### 如何进行设置worker数量

在配置文件中修改即可

worker_processes 4	# 表示4个worker进程

worker_cpu_affinity 0001 0010 0100 1000	#4个worker绑定4个cpu



worker_cpu_affinity 00000001 00000010 00000100 00001000	# 4个worker绑定8cpu中的4个



### 连接数worker_connection

这个值是表示每个worker进程所能建立连接的最大值，因此，一个nginx能建立的最大连接数，应该是worker_connections*worker_processes。当然，这里说的是最大连接数，对于Http请求本地资源来说，能支持的最大并发数量是worker_connection\*worker_processes，如果是支持http1.1的浏览器，每次访问要占用两个连接，所以普通的静态访问最大并发数是：worker_connection\*worker_processes/2，而如果是HTTP做反向代理，最大并发数量应该是worker_connection\*worker_processes/4,。因为作为反向代理服务器，每个并发会建立与客户端的连接以及后端服务器的连接，所以总共会占用4个连接。



