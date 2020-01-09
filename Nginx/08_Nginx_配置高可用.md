---
title: 08_Nginx_配置高可用
date: 2020-01-09 20:34:23
tags: 
 - Nginx
categories:
 - Nginx
---

# 08_Nginx_配置高可用

![image-20200109204337291](08_Nginx_%E9%85%8D%E7%BD%AE%E9%AB%98%E5%8F%AF%E7%94%A8/image-20200109204337291.png)

1. 高可用前提

   - 两台Nginx服务器
   - 需要keepalived
   - 虚拟的ip地址

2. 配置高可用的准备工作

   - 需要两台服务器
   - 两台服务器上安装Nginx
   - 在两台服务器上安装keepalived
     - 使用yum安装`yum install keepalived -y`
     - 查看是否安装上`rpm -q -a keepalived`
     - 配置文件位置在/etc/keepalived/keepalived.conf

3. 完成高可用配置（主从）

   keepalived.conf：

   ```shell
   global_defs {
   	notification_email {
   		acassen@firewall.loc
   		failover@firewall.loc
   		sysadmin@firewall.loc
   	}
   	notification_email_from Alexandre.Cassen@firewall.loc
   	smtp_server 192.168.17.129
   	smtp_connect_timeout 30
   	router_id LVS_DEVEL	# 访问到主机的名字，在etc/hosts中添加127.0.0.1 LVS_DEVEL
   }
   
   vrrp_script chk_http_port {
   	script "/usr/local/src/nginx_check.sh"
   	interval 2 # (检测脚本执行的间隔)
   	weight 2	# 设置当前服务器的权重
   }
   
   vrrp_instance VI_1 {
   	state BACKUP	# 备份服务器上将MASTER 改为BACKUP
   	interface ens33	# 网卡名称
   	virtual_router_id 51	# 主、备机的virtual_router_id必须相同
   	priority 100	# 主、备机取不同的优先级，主机值比较大，备份机比较小
   	advert_int 1	# 心跳间隔 默认1秒
   	authentication {
   		auth_type PASS	# 密码的方式
   		auth_pass 1111	# 密码
   	}
   	virtual_ipaddress{
   		192.168.17.50	#  VRRP  H虚拟地址，可以绑定多个
   	}
   }
   ```

   第一部分的global_defs，全局定义块，主要关注router_id ，这个值是唯一的；

   第二部分是检测脚本以及权重的参数，script指向脚本的位置，脚本需要另外编写

   脚本：

   ```shell
   # !/bin/bash
   A=`ps -C nginx -no-header | wc -l`
   if [ $A -eq 0];then
   	/usr/local/nginx/sbin/nginx
   	sleep 2
   	if [ `ps -C nginx --no-header | wc -l` -eq 0 ];then
   		killall keepalived
   	fi
   fi
   ```

4. 启动两台服务器的Nginx以及keepalived

   - 可以用ip a看网卡绑定虚拟ip情况

   - 启动nginx：`./nginx`

   - 启动keepalived：`systemctl start keepalived.service`

5. 最终测试

   - 先正常访问**虚拟IP**，再把Nginx主服务器停掉，然后再访问，看是否还能访问到

