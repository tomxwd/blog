---
title: 08_SpringBoot_Docker
date: 2019-07-11 20:15:38
tags: 
 - Java
 - SpringBoot
categories:
 - Spring
 - SpringBoot初学
---

# 08_SpringBoot_Docker

![Docker简介](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/SpringBoot/08Docker%E7%AE%80%E4%BB%8B.png)

简介：

- Docker是一个开源的应用容器引擎；

- Docker支持将软件编译成一个镜像，然后在镜像中各种软件做好配置，将镜像发布出去，其他使用者可以直接使用这个镜像；

- 运行中的这个镜像成为容器，容器的启动是非常快速的。



## Docker核心概念

![Docker核心概念](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/SpringBoot/08Docker%E6%A0%B8%E5%BF%83%E6%A6%82%E5%BF%B5.png)

- docker主机（Host）：安装了Docker程序的机器（Docker直接安装在操作系统之上）

- docker客户端（Client）：连接docker主机进行操作；

- docker仓库（Registry）：用来保存各种打包好的软件镜像；

- docker镜像（Images）：软件打包好的镜像，放在docker仓库中；

- docker容器（Container）：镜像启动后的实例成为一个容器；容器是独立运行的一个或一组应用；



**使用Docker的步骤：**

1. 安装Docker
2. 去Docker的仓库找到这个软件对应的镜像
3. 使用Docker运行这个镜像，这个镜像就会生成一个Docker容器
4. 对容器的启动停止就是对软件的启动停止



## Docker的安装

### 安装Linux虚拟机

1. VMware（太重量级了）
2. **VirtualBox**（Oracle免费的虚拟机软件，小巧）

这里用VirtualBox，点开安装完，直接导入虚拟机文件（ova格式），点上重置网卡选项。

然后安装SmarTTY客户端软件。

此时还没能连上虚拟机，需要**设置虚拟机的网络**

1. 桥接网络
2. 选好网卡
3. 点上接入网线
4. service network restart（centos 7）或重启虚拟机
5. 查看虚拟机IP地址（`ip addr`）

使用SmarTTY连接即可使用。



### 安装Docker

![Docker安装](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/SpringBoot/08Docker%E5%AE%89%E8%A3%85.png)

//TODO





## 容器操作

![docker常用操作](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/SpringBoot/08docker%E5%B8%B8%E7%94%A8%E6%93%8D%E4%BD%9C.png)



**软件镜像**（类似QQ安装程序）----运行镜像----产生一个容器（正在运行的软件，类比启动QQ.exe）

![容器操作](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/SpringBoot/08%E5%AE%B9%E5%99%A8%E6%93%8D%E4%BD%9C.png)

步骤：

1. 搜索镜像

   ```shell
   [root@izwz92w1juq9po3gy6wrk7z ~]# docker search tomcat
   ```

2. 拉取镜像

   ```shell
   [root@izwz92w1juq9po3gy6wrk7z ~]# docker pull tomcat
   ```

3. 根据镜像启动容器

   ```shell
   [root@izwz92w1juq9po3gy6wrk7z ~]# docker run --name mytomcat -d tomcat:latest
   ```

4. 查看运行中的容器

   ```shell
   [root@izwz92w1juq9po3gy6wrk7z ~]# docker ps
   ```

   由于没有映射端口，所以访问不到tomcat。

5. 停止运行中的容器

   ```shell
   [root@izwz92w1juq9po3gy6wrk7z ~]# docker stop mytomcat
   ```

   可以用容器id来停止。

6. 查看所有的容器

   ```shell
   [root@izwz92w1juq9po3gy6wrk7z ~]# docker ps -a
   ```

7. 启动容器

   ```shell
   [root@izwz92w1juq9po3gy6wrk7z ~]# docker start tomcat
   ```

8. 删除容器

   ```shell
   [root@izwz92w1juq9po3gy6wrk7z ~]# docker rm mytomcat
   ```

9. 映射端口并运行tomcat容器

   ```shell
   [root@izwz92w1juq9po3gy6wrk7z ~]# docker run --name mytomcat -d -p 8888:8080 tomcat
   ```

   -d：后台运行

   -p：将主机的端口映射到容器的一个端口 				主机端口:容器内部端口

10. 注意：要访问到这个Tomcat，要么就开放指定端口，要么就关了防火墙，否则外部无法访问。

    查看防火墙的状态：

    ```shell
    [root@izwz92w1juq9po3gy6wrk7z ~]# service firewalld status
    ```

    关闭防火墙：

    ```shell
    [root@izwz92w1juq9po3gy6wrk7z ~]# service firewalld stop
    ```

11. 查看docker的日志

    ```shell
    [root@izwz92w1juq9po3gy6wrk7z ~]# docker logs mytomcat
    ```

直接看官方说明文档会有详细的配置说明。



## 环境搭建

1. 安装MySQL
2. 安装Redis
3. 安装RabbitMQ
4. 安装ElasticSearch



### 安装mysql示例

```shell
docker pull mysql
```

安装最新的mysql。



这是错误的启动：

```shell
[root@izwz92w1juq9po3gy6wrk7z ~]# docker run --name springboot-mysql -d mysql
47b024bc5914fd5f8b4528d2c921a66289b0952d3aa82e7346e6fc7d4df33a69
```

`docker ps`查看运行中的容器，并没有springboot-mysql

而`docker ps -a`查看所有容器，发现springboot-mysql是exit状态。mysql退出了。



查看docker的日志，用`docker logs springboot-mysql`:

```shell
[root@izwz92w1juq9po3gy6wrk7z ~]# docker logs springboot-mysql
error: database is uninitialized and password option is not specified 
  You need to specify one of MYSQL_ROOT_PASSWORD, MYSQL_ALLOW_EMPTY_PASSWORD and MYSQL_RANDOM_ROOT_PASSWORD
```

错误日志：数据库在初始化的时候密码项没有指定，我们必须指定这里面其中一个参数（`MYSQL_ROOT_PASSWORD  ,  MYSQL_ALLOW_EMPTY_PASSWORD  ,  MYSQL_RANDOM_ROOT_PASSWORD`）。



正确的启动（应该到docker-hub查看官方的文档进行启动）：

```shell
[root@izwz92w1juq9po3gy6wrk7z ~]# docker run --name springboot-mysql -e MYSQL_ROOT_PASSWORD=root -d mysql;
```

官网上看到的比较正确的启动命令。但是发现还差个端口映射没配。

重新整一份：

```shell
[root@izwz92w1juq9po3gy6wrk7z ~]# docker run --name springboot-mysql -p 12389:3306 -e MYSQL_ROOT_PASSWORD=root -d mysql;
```

即可尝试连接：

报错：

![错误提示](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/SpringBoot/08%E9%94%99%E8%AF%AF%E6%8F%90%E7%A4%BA.png)

mysql8.0默认使用caching_sha2_password身份验证机制；客户端并不支持新的加密方式

解决方案：

修改用户（root）的加密方式：

1. 进入mysql

   ```shell
   [root@izwz92w1juq9po3gy6wrk7z ~]# docker exec -it springboot-mysql bash
   ```

2. 登录mysql

   ```shell
   root@fe1861f43b33:/# mysql -u root -p     
   Enter password: 
   Welcome to the MySQL monitor.  Commands end with ; or \g.
   Your MySQL connection id is 12
   Server version: 8.0.16 MySQL Community Server - GPL
   
   Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.
   
   Oracle is a registered trademark of Oracle Corporation and/or its
   affiliates. Other names may be trademarks of their respective
   owners.
   
   Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
   
   mysql> 
   ```

3. 设置用户配置项

   1. 查看用户信息

      ```shell
      mysql> select host,user,plugin,authentication_string from mysql.user;
      +-----------+------------------+-----------------------+------------------------------------------------------------------------+
      | host      | user             | plugin                | authentication_string                                                  |
      +-----------+------------------+-----------------------+------------------------------------------------------------------------+
      | %         | root             | caching_sha2_password | $A$005$L-KWV|s}U=<r{rHpTU/NoyY7v1/4n425RQHZqkcYIcqJujE4Z7h4kIfuoA |
      | localhost | mysql.infoschema | caching_sha2_password | $A$005$THISISACOMBINATIONOFINVALIDSALTANDPASSWORDTHATMUSTNEVERBRBEUSED |
      | localhost | mysql.session    | caching_sha2_password | $A$005$THISISACOMBINATIONOFINVALIDSALTANDPASSWORDTHATMUSTNEVERBRBEUSED |
      | localhost | mysql.sys        | caching_sha2_password | $A$005$THISISACOMBINATIONOFINVALIDSALTANDPASSWORDTHATMUSTNEVERBRBEUSED |
      | localhost | root             | caching_sha2_password | $A$005$>gKE+&?.jtS_WFv
                                                                                     oo77e2AccPJtYR3rHsSutMTVg.I4YetUG7w2tWhQmk2 |
      +-----------+------------------+-----------------------+------------------------------------------------------------------------+
      5 rows in set (0.00 sec)
      ```

      这里可以看到：host是%表示不限制ip，localhost表示本机，那么%使用的plugin不是mysql_native_password，则需要改成mysql_native_password。

   2. 修改加密方式并刷新权限

      ```shell
      mysql> ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY 'root';
      Query OK, 0 rows affected (0.01 sec)
      
      mysql> flush privileges;
      Query OK, 0 rows affected (0.00 sec)
      ```

   3. 再次查看用户信息

      ```shell
      mysql> select host,user,plugin,authentication_string from mysql.user;
      +-----------+------------------+-----------------------+------------------------------------------------------------------------+
      | host      | user             | plugin                | authentication_string                                                  |
      +-----------+------------------+-----------------------+------------------------------------------------------------------------+
      | %         | root             | mysql_native_password | *81F5E21E35407D884A6CD4A731AEBFB6AF209E1B                              |
      | localhost | mysql.infoschema | caching_sha2_password | $A$005$THISISACOMBINATIONOFINVALIDSALTANDPASSWORDTHATMUSTNEVERBRBEUSED |
      | localhost | mysql.session    | caching_sha2_password | $A$005$THISISACOMBINATIONOFINVALIDSALTANDPASSWORDTHATMUSTNEVERBRBEUSED |
      | localhost | mysql.sys        | caching_sha2_password | $A$005$THISISACOMBINATIONOFINVALIDSALTANDPASSWORDTHATMUSTNEVERBRBEUSED |
      | localhost | root             | caching_sha2_password | $A$005$>gKE+&?.jtS_WFv
                                                                                     oo77e2AccPJtYR3rHsSutMTVg.I4YetUG7w2tWhQmk2 |
      +-----------+------------------+-----------------------+------------------------------------------------------------------------+
      5 rows in set (0.00 sec)
      ```

      此时，host为%的用户root，plugin为mysql_native_password，再次连接试试。

4. 再次连接

   ![MySQL成功连接](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/SpringBoot/08MySQL%E6%88%90%E5%8A%9F%E8%BF%9E%E6%8E%A5.png)

   可行。

   

几个其他高级操作：

指定配置文件：

```shell
$ docker run --name some-mysql -v /my/custom:/etc/mysql/conf.d -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag
```

-v：把主机的/my/custom文件夹挂载到mysql容器的/etc/mysql/conf.d文件夹里面，改mysql的配置文件，就只需要把mysql配置文件放在/my/custom，官方文档说，会将合并里面的文件。



不使用配置文件：

```shell
$ docker run --name some-mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
```

--Xxx 指定配置mysql的一些参数，这里改变字符集配置为utf-8；



我们重新生成一个容器，加上`--character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci`，改变其字符集，方便以后的操作，以免中文乱码：

```shell
[root@izwz92w1juq9po3gy6wrk7z ~]# docker run --name springboot-mysql -d -p 12389:3306 -e MYSQL_ROOT_PASSWORD=root mysql --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
```

此时还是要去改用户root的加密方式，过程如上。



其他的容器安装自己去看docker-hub即可，可以尝试安装。



