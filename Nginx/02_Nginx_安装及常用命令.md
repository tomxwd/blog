---
title: 02_Nginx_安装及常用命令
date: 2020-01-05 21:11:48
tags: 
 - Nginx
categories:
 - Nginx
---

# 02_Nginx_安装及常用命令

> [Nginx官网](http://nginx.org)

1. 连接Linux服务器

2. [官网](http://nginx.org)下载1.12.2版本Nginx

3. Nginx需要的相关素材（依赖）

   - pcre-8.37.tar.gz

     ```shell
     wget http://downloads.sourceforge.net/project/pcre/pcre/8.37/pcre-8.37.tar.gz
     ```

     解压文件`tar -zxvf pcre-8.37.tar.gz`，在./configure完成后，回到pcre目录下执行make，最后执行make install

     安装之后，使用命令，查看版本号`pcre-config -version`

   - openssl-1.0.1t.tar.gz

   - zlib-1.2.8.tar.gz

     ```shell
     yum -y install make zlib zlib-devel gcc-c++ libtool openssl openssl-devel
     ```

   - nginx-1.12.2.tar.gz

   **一键安装上面四个依赖：**

   ```shell
   // 一键安装四个依赖
   yum -y install gcc zlib zlib-devel pcre-devel openssl openssl-devel
   ```

4. 安装Nginx

   - 解压nginx.xx.tar.gz包
   - 进入解压缩目录，执行./configure
   - make && make install
   - 安装成功之后，在usr/local会多出一个文件夹nginx，在nginx中有sbin有启动脚本，在sbin目录下，用./nginx启动Nginx；  

5. 查看开放的端口号

   - firewall-cmd --list-all

6. 设置开放的端口号

   - firewall-cmd --add-service=http --permanent【开启http服务】
   - sudo firewall-cmd --add-port=80/tcp --permanent

7. 重启防火墙

   - firewall-cmd --reload



## Nginx操作的常用命令

使用Nginx操作命令的前提条件，必须进入nginx的目录【/usr/local/nginx/sbin】

1. 查看nginx的版本号
   - ./nginx -v
2. 启动Nginx
   - ./nginx
3. 关闭Nginx
   - ./nginx -s stop：强制退出
   - ./nginx -s quit：普通退出
4. 重新加载Nginx
   - ./nginx -s reload

