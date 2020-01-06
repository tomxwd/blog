---
title: 03_Nginx_配置文件
date: 2020-01-05 22:46:42
tags: 
 - Nginx
categories:
 - Nginx
---

# 03_Nginx_配置文件

**Nginx配置文件位置：**/usr/local/nginx/conf文件夹下的nginx.conf，修改前最好备份一份；或者在/etc目录下

## Nginx配置文件组成

```shell
#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            index  index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }


    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}

}
```



1. Nginx配置文件由三个部分组成
   - **第一部分：全局块**
   
     从配置文件到events块之间的内容，主要会设置一些影响nginx服务器整体运行的配置指令，主要包括配置运行Nginx服务器的用户（组）、允许生成的worker process数，进程PID存放路径、日志存放路径和类型，以及配置文件的引入等。
   
     比如第一行的配置：
   
     ```shell
     worker_processes 1;
     ```
   
     这是Nginx服务器并发处理服务的关键配置，worker_processes值越大，可以支持的并发处理量也越多，但是会受到硬件、软件等设备的制约；
   
   - **第二部分：events块**
   
     比如上面的配置：
   
     ```shell
     events {
         worker_connections  1024;
     }
     ```
   
     events块设计的指令主要影响**Nginx服务器与用户的网络连接**，常用的设置包括是否开启对多work process下的网络连接进行序列化，是否允许同时接收多个网络连接，选取哪种事件驱动模型来处理连接请求，每个work process可以同时支持的最大连接数等。
   
     **上述例子就表示work process支持的最大连接数为1024**
   
     这部分配置对Nginx的性能影响较大，在实际中应该灵活配置
   
   - **第三部分：http块**
   
     ```shell
     http {
         include       mime.types;
         default_type  application/octet-stream;
     
         #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
         #                  '$status $body_bytes_sent "$http_referer" '
         #                  '"$http_user_agent" "$http_x_forwarded_for"';
     
         #access_log  logs/access.log  main;
     
         sendfile        on;
         #tcp_nopush     on;
     
         #keepalive_timeout  0;
         keepalive_timeout  65;
     
         #gzip  on;
     
         server {
             listen       80;
             server_name  localhost;
     
             #charset koi8-r;
     
             #access_log  logs/host.access.log  main;
     
             location / {
                 root   html;
                 index  index.html index.htm;
             }
     
             #error_page  404              /404.html;
     
             # redirect server error pages to the static page /50x.html
             #
             error_page   500 502 503 504  /50x.html;
             location = /50x.html {
                 root   html;
             }
     
             # proxy the PHP scripts to Apache listening on 127.0.0.1:80
             #
             #location ~ \.php$ {
             #    proxy_pass   http://127.0.0.1;
             #}
     
             # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
             #
             #location ~ \.php$ {
             #    root           html;
             #    fastcgi_pass   127.0.0.1:9000;
             #    fastcgi_index  index.php;
             #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
             #    include        fastcgi_params;
             #}
     
             # deny access to .htaccess files, if Apache's document root
             # concurs with nginx's one
             #
             #location ~ /\.ht {
             #    deny  all;
             #}
         }
     
     
         # another virtual host using mix of IP-, name-, and port-based configuration
         #
         #server {
         #    listen       8000;
         #    listen       somename:8080;
         #    server_name  somename  alias  another.alias;
     
         #    location / {
         #        root   html;
         #        index  index.html index.htm;
         #    }
         #}
     
     
         # HTTPS server
         #
         #server {
         #    listen       443 ssl;
         #    server_name  localhost;
     
         #    ssl_certificate      cert.pem;
         #    ssl_certificate_key  cert.key;
     
         #    ssl_session_cache    shared:SSL:1m;
         #    ssl_session_timeout  5m;
     
         #    ssl_ciphers  HIGH:!aNULL:!MD5;
         #    ssl_prefer_server_ciphers  on;
     
         #    location / {
         #        root   html;
         #        index  index.html index.htm;
         #    }
         #}
     
     }
     ```
   
     这算是Nginx服务器配置中最频繁的部分，代理、缓存和日志定义等绝大多数功能和第三方模块的配置都在这里，需要注意的是：**http块也可以包括http全局块、server块**
   
     - http全局块
   
       http全局块配置的指令包括文件引入、MIME-TYPE定义、日志自定义、连接超时时间、单链接请求书数上限等；
   
     - server块
   
       - 这块和虚拟主机有密切关系，虚拟主机从用户角度看，和一台独立的硬件主机是完全一样的，该技术的产生是为了节省互联网服务器硬件成本。
   
       - 每个http块可以包括多个server块，而每个server块就相当于一个虚拟主机。
   
       - 每个server块也分为全局server块，以及可以同时包含多个location块。
   
       - **全局server块**：
   
         最常见的配置是本虚拟机主机的监听配置和本虚拟机主机的名称或IP配置。
   
       - **location块：**
   
         一个server块可以配置多个location块。
   
     这块的主要作用是基于Nginx服务器收到的请求字符串（例如server_name/uri-string），对虚拟主机名称（也可以是ip别名）之外的字符串（例如前面的uri-string）进行匹配，对特定的请求进行处理。地址定向、数据缓存和应答控制等功能，还有许多第三方模块的配置也在这里运行。

