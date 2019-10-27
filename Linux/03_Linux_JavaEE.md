---
title: 03_Linux_JavaEE
date: 2019-10-05 20:37:38
tags: 
 - Linux
categories:
 - Linux
---

# 03_Linux_JavaEE

## 安装JDK

> [下载地址](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)
>
> 需要账号密码，百度一下就有
>
> 下载-x64.tar.gz版本

1. 传jdk到/opt/open/download目录下

2. 解压到/opt/open/environment/jdk/目录下

   - 注意，如果是解压到指定目录，要加上-C参数

   - tar -zxvf jdk-8u221-linux-x64.tar.gz -C /opt/open/environment/jdk/

3. 配置环境变量配置文件：vim /etc/profile

4. JAVA_HOME=/opt/open/environment/jdk/jdk1.8.0_221

5. PATH=/opt/open/environment/jdk/jdk1.8.0_221/bin:$PATH

6. export JAVA_HOME PATH

7. 直接生效：source /etc/profile，如果不这样就注销用户再进入

8. 测试是否可用，先用java -version，再编写一个HelloWorld编译后运行试试。

## 安装tomcat

> [tomcat8下载地址](https://tomcat.apache.org/download-80.cgi)

1. 放到/opt/open/download目录下

2. 解压到/opt/open/service/tomcat/目录下

   - tar -zxvf apache-tomcat-8.5.46.tar.gz -C /opt/open/service/tomcat/

3. 启动tomcat ./startup.sh

4. 开放端口vim /etc/sysconfig/iptables

   - 复制22端口的形式，改成8080即可

   - -A INPUT -m state --state NEW -m tcp -p tcp --dport 8080 -j ACCEPT

5. 重启防火墙

   - service iptables restart

6. 测试windows是否可以访问到linux上的tomcat服务器

## 安装eclipse

1. 放到/opt/download目录下
2. 解压到/opt/soft/eclipse目录下
3. 启动eclipse，配置jre和server
4. 编写HelloWorld程序并测试
5. 编写jsp页面并测试

## 安装mysql5.6.44

> [mysql官方下载地址](https://downloads.mysql.com/archives/community/)
>
> [网上步骤](https://blog.csdn.net/hwh1231/article/details/71426844)
>
> 记得要下载**Source Code**版本的mysql，因为这里是源码安装

1. 先卸载：

   rpm -qa|grep mysql

   如果有查到，就直接删除：

   - rpm -e mysql：普通删除
   - rpm -e --nodeps mysql ：强制删除，如果上面的命令提示有依赖，可以用这个命令进行强制删除

2. 安装编译代码需要的包

   - yum -y install make gcc-c++ cmake bison-devel ncurses-devel

3. 下载MySQL5.6.44

4. 解压

   - tar -zxvf mysql-5.6.44.tar.gz

5. 进入到mysql-5.6目录

6. 编译安装

   - ```shell
     cmake -DCMAKE_INSTALL_PREFIX=/usr/local/mysql -DMYSQL_DATADIR=/usr/local/mysql/data -DSYSCONFDIR=/etc -DWITH_MYISAM_STORAGE_ENGINE=1 -DWITH_INNOBASE_STORAGE_ENGINE=1 -DWITH_MEMORY_STORAGE_ENGINE=1 -DWITH_READLINE=1 -DMYSQL_UNIX_ADDR=/var/lib/mysql/mysql.sock -DMYSQL_TCP_PORT=3306 -DENABLED_LOCAL_INFILE=1 -DWITH_PARTITION_STORAGE_ENHINE=1 -DEXTRA_CHARSETS=all -DDEFAULT_CHARSET=utf8 -DDEFAULT_COLLATION=utf8_general_ci
     ```
     
   - ```shell
     cmake . -DCMAKE_INSTALL_PREFIX=/data/mysql/installdir \
     -DDEFAULT_CHARSET=utf8 \
     -DDEFAULT_COLLATION=utf8_general_ci \
     -DENABLED_LOCAL_INFILE=ON \
     -DWITH_INNOBASE_STORAGE_ENGINE=1 \
    -DWITH_FEDERATED_STORAGE_ENGINE=1 \
     -DWITH_BLACKHOLE_STORAGE_ENGINE=1 \
     -DWITHOUT_EXAMPLE_STORAGE_ENGINE=1 \
     -DWITH_PARTITION_STORAGE_ENGINE=1 \
     -DWITH_PERFSCHEMA_STORAGE_ENGINE=1 \
     -DCOMPILATION_COMMENT='MySQL' \
     -DWITH_READLINE=ON \
     -DWITH_BOOST=/data/mysql/src/mysql-5.7.18/boost \
     -DSYSCONFDIR=/data/mysql/datadir/3306/data \
     -DMYSQL_UNIX_ADDR=/data/mysql/datadir/3306/data/mysql.sock
     ```
   
   - make && make install
   
7. 配置MySQL

   - 设置权限
     - 查看是否有mysql用户和mysql组
       - cat /etc/passwd：查看用户列表
       - cat /etc/group：查看用户组列表
     - 无则创建：
       - groupadd mysql
       - useradd -g mysql mysql
     - 修改/usr/local/mysql权限
       - chown -R mysql:mysql /user/local/mysql（-R表示递归修改权限）
   - 初始化配置，进入安装路径（再执行下面的指令），执行初始化配置脚本，创建系统自带的数据库和表
     - cd /usr/local/mysql
     - scripts/mysql_install_db --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data --user=mysql
   - **注意：**
     - **在启动MySQL服务的时候，会按照一定次序搜查my.cnf配置文件，先在/etc目录下找，找不到就会搜索"$basedir/my.cnf"，这里就是/usr/local/mysql/my.cnf，这是最新版MySQL的配置文件的默认位置！**
     - **在CentOS6.8版本操作系统的最小安装完成后，会在/etc目录下有个my.cnf，需要吧这个文件改名，如/etc/my.cnf.bak，否则该文件会干扰源码安装的MySQL正确配置，造成无法启动**

8. 启动MySQL

   - 添加服务，拷贝服务脚本到init.d目录，并设置为开机启动
     - cd到/usr/local/mysql路径下
     - cp support-files/mysql.server /etc/init.d/mysql（复制服务到/etc/init.d/mysql目录）
     - chkconfig mysql on（设置为开机启动）
     - service mysql start（启动MySQL）
       - 如果不能启动，查看日志，可能是因为/var/lib/mysql没有给mysql用户权限

9. 修改root密码

   - cd /usr/local/mysql/bin
   - ./mysql -uroot -p
   - mysql > SET PASSWORD = PASSWORD('root');

10. 可以把bin放到环境变量里头去（vim /etc/profile）

    - PATH=/usr/local/mysql/bin:/.....:$PATH
    - source /etc/profile

11. 记得要开放防火墙：

    - /etc/sysconfig/iptables
    - service iptables restart

12. navicat远程连接可能失败

    - 可以进入命令行后，update user set host='%' where user = 'root';
    - 然后flush  privileges;

    