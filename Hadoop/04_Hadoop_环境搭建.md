---
title: 04_Hadoop_环境搭建
date: 2019-09-29 21:31:11
tags: 
 - Hadoop
categories:
 - Hadoop
---

# 04_Hadoop_环境搭建

## 虚拟机环境准备

1. 克隆虚拟机

2. 修改克隆虚拟机的静态IP
   - vim /etc/udev/rules.d/70-persisitent-net.rules
   
     ```shell
     # This file was automatically generated by the /lib/udev/write_net_rules
     # program, run by the persistent-net-generator.rules rules file.
     #
     # You can modify it, as long as you keep each rule on a single
     # line, and change only the value of the NAME= key.
     
     # PCI device 0x8086:0x100f (e1000)
     SUBSYSTEM=="net", ACTION=="add", DRIVERS=="?*", ATTR{address}=="00:0c:29:e2:1d:3d", ATTR{type}=="1", KERNEL=="eth*", NAME="eth0"
     ```
   
     这里的物理地址需要复制到下面的配置文件HWADDR中去。
   
     如果是克隆的话，把第一块eth0删除，然后把第二块eth1改为eth0；
   
   - vim /etc/sysconfig/network-scripts/ifcfg-eth0
   
     ```shell
     DEVICE="eth0"
     BOOTPROTO="static"
     HWADDR="00:0C:29:E2:1D:3D"
     IPV6INIT="yes"
     NM_CONTROLLED="yes"
     ONBOOT="yes"
     TYPE="Ethernet"
     UUID="0b772052-7949-42c8-8467-e15c36d69b6e"
     
     IPADDR=192.168.5.101
     GATEWAY=192.168.5.2
     DNS1=192.168.5.2
     ```
     
     这里配BOOTPROTO为static，以及HWADDR为上一步复制的值。
   
3. 修改主机名

   - vim /etc/sysconfig/network

     ```shell
     NETWORKING=yes
     HOSTNAME=hadoop101
     ```

   - vim /etc/hosts

     ```shell
     127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
     ::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
     192.168.5.100 hadoop100
     192.168.5.101 hadoop101
     192.168.5.102 hadoop102
     192.168.5.103 hadoop103
     192.168.5.104 hadoop104
     192.168.5.105 hadoop105
     192.168.5.106 hadoop106
     192.168.5.107 hadoop107
     192.168.5.108 hadoop108
     ```

4. 关闭防火墙

5. 创建tomxwd用户

6. 配置tomxwd用户具有的root权限

   - vim /etc/sudoers

     ```shell
     root    ALL=(ALL)       ALL
     tomxwd  ALL=(ALL)       ALL
     ```

7. 重启reboot

8. windows上记得在C:\Windows\System32\drivers\etc配置hosts

## JDK安装

在/opt目录下创建文件夹

1. 在/opt目录下创建open文件夹（个人习惯）
   - open
     - download（存放下载的tar包等）
     - soft（软件）
     - environment（环境）
     - service（服务）
   
   sudo mkdir open
2. 修改open文件夹的所有者cd
   
   - sudo chown tomxwd:tomxwd open

把JDK拷贝到Linux上，download目录下即可。



安装步骤：

1. 解压：

   tar -zxvf jdk-8u221-linux-x64.tar.gz -C /opt/open/environment/

2. 获取安装路径：

   pwd

   /opt/open/environment/jdk1.8.0_221

3. 配置环境变量：

   sudo vim /etc/profile

   末尾追加以下内容

   ```shell
   ## JAVA_HOME
   export JAVA_HOME=/opt/open/environment/jdk1.8.0_221
   export PATH=$PATH:$JAVA_HOME/bin
   ```

4. source /etc/profile

5. java -version看是否安装成功



## Hadoop安装

> [下载地址1](http://archive.apache.org/dist/hadoop/core/)
>
> [下载地址2](http://hadoop.apache.org/releases.html)
>
> [清华的下载地址3,](https://mirrors.tuna.tsinghua.edu.cn/apache/hadoop/common/)
>
> 下载binary版本，否则需要自己去编译

把hadoop拷贝到Linux上，download目录下即可。



安装步骤：

1. 解压：

   tar -zxvf hadoop-2.9.2.tar.gz -C /opt/open/environment/

2. 获取安装路径：

   pwd

   /opt/open/environment/hadoop-2.9.2

3. 配置环境变量：

   sudo vim /etc/profile

   末尾追加内容

   ```shell
   # HADOOP_HOME
   export HADOOP_HOME=/opt/open/environment/hadoop-2.9.2
   export PATH=$PATH:$HADOOP_HOME/bin
   export PATH=$PATH:$HADOOP_HOME/sbin
   ```

4. source /etc/profile

5. hadoop version看是否安装成功



跟jdk不同的就是环境变量PATH多一个sbin，以及看版本是用version而不是-version