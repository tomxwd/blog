---
title: 01_MySQL_简介
date: 2019-12-28 22:10:25
tags: 
 - MySQL
categories:
 - MySQL
---

# 01_MySQL_简介

> [相关代码GitHub仓库](https://github.com/tomxwd/MySQL-Basic)
>
> git@github.com:tomxwd/MySQL-Basic.git
>
> https://github.com/tomxwd/MySQL-Basic.git

## 简介

DB：数据库（Database），存储数据的仓库，保存一系列有组织的数据；

DBMS：数据库管理系统（Database Management System），数据库是通过DBMS创建和操作的容器

- 常见的数据库管理系统：MySQL、Oracle、DB2、SQLServer等；

SQL：结构化查询语言（Structure Query Language），专门用来与数据库通信的语言；



## MySQL卸载

1. 程序卸载
2. 安装所在目录卸载
3. 系统盘ProgramData/MySQL删除
4. 若没清理干净，则清理注册表：
   - HKEY_LOCAL_MACHINE\SYSTEM\ControlSet001\Services\Eventlog\Application\MySQL目录
   - HKEY_LOCAL_MACHINE\SYSTEM\ControlSet002\Services\Eventlog\Application\MySQL目录
   - HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Eventlog\Application\MySQL目录
   - HKEY_LOCAL_MACHINE\SYSTEM\CurrentControl001\Services\MySQL目录
   - HKEY_LOCAL_MACHINE\SYSTEM\CurrentControl002\Services\MySQL目录
   - HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\MySQL目录
5. 删除C:\Documents and Settings\All Users\Application Data\MySQL目录



## MySQL安装与使用

> [下载地址](https://downloads.mysql.com/archives/community/)
>
> [视频解说](https://www.bilibili.com/video/av21400736?p=7)

这里下载5.5版本；

主要配置utf-8字符集以及把mysql/bin添加到path环境变量；



## MySQL配置文件介绍

### my.ini文件介绍

[mysql]代表客户端配置

[mysqld]代表服务端配置

- port：端口号
- basedir：安装目录
- datadir：数据文件目录
- character-set-server：服务端字符集
- default-storage-engine：默认存储引擎
- sql-mode：语法格式
- max_connections：最大连接数



## MySQL服务启动与停止

1. 可以通过【服务】页面手动开启/停止/设置启动模式等；
2. 通过命令行窗口【管理员权限运行】，net stop MySQL停止服务，net start MySQL启动服务；



## MySQL服务端的登录和退出

登录：

命令：`mysql -h localhost -P 3306 -u root -proot`

- -h：host
- -P：端口号
- -u：用户名
- -p：密码，最好不显示输入，而是最后-p不带参数直接回车，再输入密码，保证密码隐藏起来；**且密码需要紧接着-p，不可以有空格；**

如果是连接本机3306的服务，只需要-u -p两个参数即可；

退出：

`exit`或者Ctrl+C；



## MySQL环境变量配置

windows端：

如果安装的时候忘了勾选，或者其他因素，可能会导致在命令行窗口显示mysql不是合法的命令，这时候就需要自己手动配置环境变量，把MySQL下的bin目录配置到Path环境变量中即可；



## MySQL的常见命令

1. 查看数据库列表

   `show databases;`

   - mysql数据库：用于保存用户信息
   - information_schema：保存元数据信息
   - performance_schema：搜集性能信息，性能参数
   - test：测试数据库，默认是空的

2. 打开指定database数据库

   `use database;`

3. 查看当前数据库的表列表

   `show tables;`

4. 查看来自database数据库的表列表

   `show tables from database;`

5. 查看当前所在数据库

   `select database();`

6. 创建stuinfo表

   `create table stuinfo(id int,name varchar(20));`

7. 查看表结构

   `desc stuinfo;`

8. 查看MySQL版本

   `select version();`

   或者执行bin目录下的mysql，`mysql --version`或者`mysql -V`；

9. 配置时区：

   - `show variables like'%time_zone';`查看系统时间信息，如果是SYSTEM则是没设置，有时候会导致图形客户端连接不上；
   - `set global time_zone = '+8:00';`设置为东八区时间即可；

10. 



### MySQL语法规范

1. 不区分大小写，建议关键字大写，表名列名小写；
2. 每条命令用分号结尾；\g也可以，但麻烦；
3. 每条命令根据需要，可以进行缩进或换行；
4. 注释
   - 单行注释
     - #
     - --空格
   - 多行注释
     - /* 注释文字 */









