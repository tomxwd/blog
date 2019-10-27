---
title: 06_Hadoop_运行模式
date: 2019-10-07 18:13:48
tags: 
 - Hadoop
categories:
 - Hadoop
---

# 06_Hadoop_运行模式

Hadoop运行模式包括：本地模式、伪分布式模式以及完全分布式模式。

[Hadoop官方网站](http://hadoop.apache.org/)

[getting start](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/SingleCluster.html)

## 本地运行模式

### 官方Grep案例

```shell
$ mkdir input
$ cp etc/hadoop/*.xml input
$ bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-3.2.1.jar grep input output 'dfs[a-z.]+'
$ cat output/*
```

1. cd到hadoop目录下

2. mkdir input

3. 复制etc下的配置文件到input

   - cp etc/haddop/*.xml input

4. 运行share目录下的案例jar，且指定为grep案例，指定input以及output，这里output不可以先创建，否则报错，然后正则一下

   - hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.9.2.jar grep input output 'dfs[a-z.]+'

5. 查看output目录

   - 两个文件，一个是_SUCCESS，这个只是标记，标记成功完成。

   - 另一个文件是结果，这里cat一下看看

     ```shell
     1	dfsadmin
     ```

     只有一个结果。

   - 可以改一下正则表达式的内容查看输出。



### 官方WordCount案例

目的：统计单词的个数

1. cd到hadoop目录下

2. 创建wcinput目录

   - mkdir wcinput

3. 在wcinput目录下创建wc.input文件

   - touch wc.input

4. 编辑wc.input文件

   ```shell
   hadoop yarn
   hadoop mapreduce
   tomxwd
   tomxwd
   ```

5. 回到hadoop目录

6. 运行share目录下的案例jar，指定wordcount案例，指定wcinput和wcoutput

   - hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.9.2.jar wordcount wcinput wcoutput

7. 查看执行结果

   ```
   hadoop yarn
   hadoop mapreduce
   tomxwd
   tomxwd
   ```

   

## 伪分布式模式

### 启动HDFS并运行MapReduce程序

> [官网文档](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/SingleCluster.html#Pseudo-Distributed_Operation)
>
> 在官网右侧，可以找到Configuration，是关于xml配置文件的详细解释

伪分布式模式的配置信息与完全分布式模式的一样，只是说伪分布式模式只用一台服务器来完成，适合学习的时候用。

1. 修改etc/hadoop/core-site.xml配置文件

   ```xml
   <!-- 指定HDFS中NameNode的地址 -->
   <property>
       <name>fs.defaultFS</name>
       <value>hdfs://localhost:9000</value>
   </property>
   <!-- 指定Hadoop运行时产生文件的存储目录，默认是在/tmp/hadoop-${user.name} -->
   <property>
       <name>hadoop.tmp.dir</name>
       <value>/opt/open/environment/hadoop-2.9.2/data/tmp</value>
   </property>
   ```

2. 修改etc/hadoop/hdfs-site.xml配置文件

   默认值是3

   ```xml
   <!-- 指定HDFS副本的数量 -->
   <property>
       <name>dfs.replication</name>
       <value>1</value>
   </property>
   ```

3. 修改etc/hadoop/hadoop-env.sh文件的jdk

   ```shell
   # Set Hadoop-specific environment variables here.
   
   # The only required environment variable is JAVA_HOME.  All others are
   # optional.  When running a distributed configuration it is best to
   # set JAVA_HOME in this file, so that it is correctly defined on
   # remote nodes.
   
   # The java implementation to use.
   JAVA_HOME=/opt/open/environment/jdk1.8.0_221
   export JAVA_HOME=${JAVA_HOME}
   ```

   上面的说明写了，指定JAVA_HOME，其他的不用

4. 启动集群

   - 格式化NameNode（第一次启动时格式化，以后就不要总格式化）

     bin/hdfs namenode -format

     如果格式化遇到任何问题，可能是data文件没有删除，日志文件没有删除

   - 启动NameNode

     sbin/hadoop-daemon.sh start namenode

     - 出现提示：starting namenode, logging to /opt/open/environment/hadoop-2.9.2/logs/hadoop-tomxwd-namenode-hadoop101.out

   - 启动DataNode

     sbin/hadoop-daemon.sh start datanode

     - 出现提示：starting datanode, logging to /opt/open/environment/hadoop-2.9.2/logs/hadoop-tomxwd-datanode-hadoop101.out

5. 查看集群

   - 查看是否启动成功
     - 可以用jps命令来查看进程，是java ps的意思，有NameNode进程，以及DataNode进程
     - http://192.168.5.101:50070是访问地址，主要关注最后的Utilities>>Browse Directory

6. 添加目录：

   - bin/hdfs dfs -mkdir -p /user/tomxwd/input

7. 查看新添加的目录：

   - bin/hdfs dfs -ls -R /

8. 把本地wcinput上传到HDFS上：

   - bin/hdfs dfs -put wcinput/wc.input /user/tomxwd/input

9. 执行一下wordcount案例：

   - bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.9.2.jar wordcount /user/tomxwd/input /user/tomxwd/output

10. 到页面上看是否有output，然后下载下来看结果，或者用bin/hdfs dfs -cat /user/tomxwd/output/p*看结果即可。



#### NameNode格式化注意事项

格式化前的工作：

1. 在格式化前，先用jps看看是否已经启动了NameNode，如果启动了要结束进程。

2. 然后再看data和log文件，如果有，则删掉。

3. 最后进行NameNode格式化。



为什么不能一直格式化NameNode？

- cd到data/tmp/dfs/name/current，查看VERSION文件的内容，以及到data/tmp/dfs/data/current，查看VERSION的内容。
  - 关注clusterID的值

- 因为格式化NameNode，会产生新的集群id，导致NameNode和DataNode的集群id不一致，集群找不到以往的数据，所以在格式化NameNode的时候，一定要删除data数据和log日志，然后再格式化NameNode

![1570547884705](06_Hadoop_%E8%BF%90%E8%A1%8C%E6%A8%A1%E5%BC%8F/1570547884705.png)



#### 查看日志

出现问题的时候看datanode.log文件，也可以看看namenode.log的信息。



### 启动YARN并运行MapReduce程序

分析

- 配置集群在YARN上运行MR
- 启动、测试集群**增删查**
- 在YARN上执行WordCount案例



执行步骤

1. 配置集群

   - 配置yarn-env.sh

     配置一下JAVA_HOME

     ```shell
     # some Java parameters
     export JAVA_HOME=/opt/open/environment/jdk1.8.0_221
     ```

   - 配置yarn-site.xml

     ```xml
     <!-- Reducer获取数据的方法 -->
     <property>
         <name>yarn.nodemanager.aux-services</name>
         <value>mapreduce_shuffle</value>
     </property>
     <!-- 指定YARN的ResourceManager的地址 -->
     <property>
         <name>yarn.resourcemanager.hostname</name>
         <value>hadoop101</value>
     </property>
     ```

   - 配置mapred-env.sh

     配置一下JAVAHOME

     ```shell
     # some Java parameters
     export JAVA_HOME=/opt/open/environment/jdk1.8.0_221
     ```

   - 配置mapred-site.xml（把mapred-site.xml.template重命名）

     `mv mapred-site.xml.template mapred-site.xml`

     `vim mapred-site.xml`

     ```xml
     <!-- 指定MR运行在YARN上 -->
     <property>
         <name>mapreduce.framework.name</name>
         <value>yarn</value>
     </property>
     ```

2. 启动集群

   - 启动前确保NameNode和DataNode已经启动

   - 启动ResourceManager

     `sbin/yarn-daemon.sh start resourcemanager`

   - 启动NodeManager

     `sbin/yarn-daemon.sh start nodemanager`

3. 集群操作

   - YARN的浏览器页面查看

     http://192.168.5.101:8088/cluster

4. 删掉之前的output，然后试着再次运行wordcount案例

   `hdfs dfs -rm -r /user/tomxwd/output`

   `hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.9.2.jar wordcount /user/tomxwd/input /user/tomxwd/output`

5. 在运行的时候查看浏览器页面



#### 配置历史服务器

为了查看程序的历史运行情况，需要配置一下历史服务器。具体步骤：

1. 配置mapred-site.xml

   ```xml
   <!-- 历史服务器端地址  -->
   <property>
       <name>mapreduce.jobhistory.address</name>
       <value>hadoop101:10020</value>
   </property>
   <!-- 历史服务器web端地址  -->
   <property>
       <name>mapreduce.jobhistory.webapp.address</name>
       <value>hadoop101:19888</value>
   </property>
   ```

2. 启动历史服务器

   `sbin/mr-jobhistory-daemon.sh start historyserver`

3. 用jps查看历史服务器是否启动

4. 查看JobHistory

   http://192.168.5.101:19888/jobhistory




#### 配置日志的聚集

日志聚集概念：应用运行完成后，将程序的运行日志信息上传到HDFS系统上；

日志聚集的好处：可以方便查看到程序运行的详情，方便开发调试；

注意：**开启日志聚集功能，需要重新启动NodeManager、ResourceManager和HistoryManager。**

​	用jps看开启了哪些，用原来启动的指令，改为stop即可。



步骤：

1. 配置yarn-site.xml

   ```xml
   <!-- 日志聚集功能 -->
   <property>
       <name>yarn.log-aggregation-enable</name>
       <value>true</value>
   </property>
   <!-- 日志保留时间为7天 -->
   <property>
   	<name>yarn.log-aggregation.retain-seconds</name>
       <value>604800</value>
   </property>
   ```

2. 关闭NodeManager、ResourceManager和HistoryServer

   ```
   yarn-daemon.sh stop nodemanager
   yarn-daemon.sh stop resourcemanager
   mr-jobhistory-daemon.sh stop historyserver
   ```

3. 启动NodeManager、ResourceManager和HistoryServer

   ```
   yarn-daemon.sh start nodemanager
   yarn-daemon.sh start resourcemanager
   mr-jobhistory-daemon.sh start historyserver
   ```

4. 删除HDFS上存在的输出文件output

   `hdfs dfs -rm -rf /user/tomxwd/output`

5. 执行WordCount程序

   `hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.9.2.jar wordcount /user/tomxwd/input /user/tomxwd/output `

6. 查看日志

   - 可以在本地日志文件目录看
   - 可以在网页直接点程序的日志看
   - http://hadoop101:19888/jobhistory 然后点击job后面的logs



#### 配置文件说明

Hadoop配置文件分为两类：默认配置文件和自定义配置文件，只有用户想修改某一默认配置值时，才需要修改自定义配置文件，更改相应的属性值。



默认配置文件：

在官网，快速开始，左边最底下有Configuration，下面都是默认配置文件的信息。

| 要获取的默认文件   | 文件存放在Hadoop的jar包中的位置                           |
| ------------------ | --------------------------------------------------------- |
| core-default.xml   | hadoop-common-x.x.x.jar/core-default.xml                  |
| hdfs-default.xml   | hadoop-hdfs-x.x.x.jar/hdfs-default.xml                    |
| yarn-default.xml   | hadoop-yarn-common-x.x.x.jar/yarn-default.xml             |
| mapred-default.xml | hadoop-mapreduce-client-core-x.x.x.jar/mapred-default.xml |

自定义配置文件：

core-site.xml、hdfs-site.xml、yarn-site.xml、mapred-site.xml四个配置文件存放在%HADOOP_HOME/etc/hadoop这个路径，用户可以根据项目需求重新进行修改配置。



## 完全分布式

分析：

1. 准备3台客户机（关闭防火墙、静态ip、主机名称）
2. 安装JDK
3. 配置环境变量
4. 安装hadoop
5. 配置环境变量
6. 配置集群
7. 单点启动
8. 配置ssh
9. 群起并测试



### 虚拟机准备

创建hadoop102，hadoop103，hadoop104三部虚拟机。



### 编写集群分发脚本xsync

1. scp（secure copy）安全拷贝

   - scp定义：
     - scp可以实现服务器与服务器之间的数据拷贝（from server1 to server2）
   - 基本语法
     - scp 		-r		 $pdir/$fname						$user@$host:$pdir/$fname
     - 命令       递归     要拷贝的文件路径/名称        目标用户@主机:目标路径/名称
   - 案例实操
     - 在hadoop101上，将hadoop101中/opt/open目录下的软件拷贝到hadoop102上。
       - scp -r /opt/open root@hadoop102:/opt/open
     - 在hadoop103上，将hadoop101服务器上的/opt/open 目录下的软件拷贝到hadoop103上。
       - sudo scp -r tomxwd@hadoop101:/opt/open root@hadoop103:/opt/open
       - sudo scp -r tomxwd@hadoop101:/opt/open ./
     - 在hadoop103上，将hadoop101服务器上的/opt/open 目录下的软件拷贝到hadoop104上。
       - sudo scp -r tomxwd@hadoop101:/opt/open root@hadoop104:/opt/open
     - 注意要把拷贝过去的文件该权限。/etc/profile同样操作，以及最后需要source /etc/profile即可。

2. rsync远程同步工具

   rsync主要用于备份和镜像，具有速度快、避免复制相同内容和支持符号链接的优点。

   **rsync和scp的区别：**用rsync做文件的复制要比scp的速度快，rsync只对差异文件做更新，而scp是把所有文件复制过去。

   - 基本语法

     - rsync			-rvl			$pdir/$fname					$user@host:$pdir/$fname

     - 命令			  选项参数  要拷贝的文件路径/名称     目标用户@主机:目标路径/名称

     - 选项参数说明：

       | 选项 | 功能         |
       | ---- | ------------ |
       | -r   | 递归         |
       | -v   | 显示复制过程 |
       | -l   | 拷贝符号链接 |

   - 案例实操
     - 把hadoop101服务器上的/opt/open目录同步到hadoop102服务器的root用户的/opt/open目录：
       - rsync -rvl /opt/open/ root@hadoop102:/opt/open

3. xsync集群分发脚本

   - 需求：循环复制文件到所有节点的相同目录下

   - 需求分析：

     - rsync命令原始拷贝：

       rsync -rvl /opt/module	root@hadoop103:/opt

     - 期望脚本：

       xsync 要同步的文件名称

     - 说明：

       在/home/tomxwd/bin这个目录下存放的脚本，tomxwd用户可以在系统的任何地方直接运行。

   - 脚本实现

     - 在/home/tomxwd目录下创建bin目录，并在bin目录下创建xsync文件，文件内容：

       ```shell
       #!/bin/bash
       
       # 1. 获取输出参数的个数，如果没有参数，就直接退出
       pcount=$#
       echo $pcount
       echo $1
       if (($pcount==0))
       then
               echo no args;
               exit
       fi
       
       # 2. 获取文件的名称
       p1=$1
       fname=`basename $p1`
       echo fname=$fname
       
       # 3. 获取上级目录的绝对路径 cd -P是进入到软链接实际的路径
       pdir=`cd -P $(dirname $p1); pwd`
       echo pdir=$pdir
       
       # 4. 获取当前用户名称
       user=`whoami`
       
       # 5. 循环
       for((host=103; host<105; host++));do
               echo --------------hadoop$host-------------------
          rsync -rvl $pdir/$fname $user@hadoop$host:$pdir
       done
       ```
     
   - 修改脚本xsync具有执行权限
   
     chmod 777 xsync
   
     u+o也够用了
   
   - 调用脚本形式：xsync文件名称
     
          xsync /home/tomxwd/bin
      
     - 注意：如果将xsync放到/home/tomxwd/bin目录下仍然不能实现全局使用，可以将xsync移动到/usr/local/bin目录下。可以用echo $PATH命令看是否包含/home/tomxwd/bin目录即可确定能否使用。



### 集群配置

1. 集群部署规划

|      | hadoop102              | hadoop103                        | hadoop104                       |
| ---- | ---------------------- | -------------------------------- | ------------------------------- |
| HDFS | NameNode<br />DataNode | <br />DataNode                   | SecondaryNameNode<br />DataNode |
| YARN | <br />NodeManager      | ResourceManager<br />NodeManager | <br />NodeManager               |

2. 配置集群

   - 核心配置文件

     core-site.xml

     ```xml
     <!-- 指定HDFS中NameNode的地址 -->
     <property>
         <name>fs.defaultFS</name>
         <value>hdfs://hadoop102:9000</value>
     </property>
     <!-- 指定Hadoop运行时产生文件的存储目录，默认是在/tmp/hadoop-${user.name} -->
     <property>
         <name>hadoop.tmp.dir</name>
         <value>/opt/open/environment/hadoop-2.9.2/data/tmp</value>
     </property>
     ```

   - HDFS配置文件

     hadoop-env.sh

     ```shell
     export JAVA_HOME=/opt/open/environment/jdk1.8.0_221
     ```

     hdfs-site.xml

     ```xml
     <!-- 指定HDFS副本的数量 -->
     <property>
         <name>dfs.replication</name>
         <value>3</value>
     </property>
     
     <!-- 指定Hadoop辅助名称节点主机配置 -->
     <property>
         <name>dfs.namenode.secondary.http-address</name>
         <value>hadoop104:50090</value>
     </property>
     ```

   - YARN配置文件

     yarn-env.sh

     ```shell
     export JAVA_HOME=/opt/open/environment/jdk1.8.0_221
     ```

     yarn-site.xml

     ```xml
     <!-- Reducer获取数据的方法 -->
     <property>
         <name>yarn.nodemanager.aux-services</name>
         <value>mapreduce_shuffle</value>
     </property>
     <!-- 指定YARN的ResourceManager的地址 -->
     <property>
         <name>yarn.resourcemanager.hostname</name>
         <value>hadoop103</value>
     </property>
     ```

   - MapReduce配置文件

     mapred-env.sh

     ```shell
     export JAVA_HOME=/opt/open/environment/jdk1.8.0_221
     ```

     mapred-site.xml（拷贝mapred-site.xml.template得到）

     ```xml
     <!-- 指定MR运行在YARN上 -->
     <property>
         <name>mapreduce.framework.name</name>
         <value>yarn</value>
     </property>
     ```

3. 在集群上分发配置好的Hadoop配置文件

   ```shell
   xsync /opt/open/environment/hadoop-2.9.2/
   ```

4. 查看文件分发情况

   用cat指令即可。

### 集群单点启动

1. 如果是第一次启动集群，需要格式化NameNode

   前提是，jps看，没有启动任何程序。删除data以及logs文件夹，再格式化！

   ```shell
   hadoop namenode -format
   ```

2. 在hadoop102上启动NameNode

   ```shell
   hadoop-daemon.sh start namenode
   ```

   ```shell
   jps
   ```

3. 在hadoop102、hadoop103以及hadoop104上分别启动DataNode

   ```shell
   hadoop-daemon.sh start datanode
   ```

   


### 配置SSH无密登录配置

1. 配置ssh

   - 基本语法

     ssh另一台电脑的ip地址

   - ssh连接时出现Host key verification failed的解决方法

     ```shell
     [tomxwd@hadoop102 ~]$ ssh 192.168.5.102
     The authenticity of host '192.168.5.102 (192.168.5.102)' can't be established.
     RSA key fingerprint is 97:36:de:ba:15:f4:99:4c:e5:0f:0a:22:03:be:72:e9.
     Are you sure you want to continue connecting (yes/no)? yes
     Warning: Permanently added '192.168.5.102' (RSA) to the list of known hosts.
     tomxwd@192.168.5.102's password: 
     Last login: Tue Oct 15 13:24:23 2019 from 192.168.5.1
     ```

     输入yes并输入密码即可

2. 无密钥配置

   - 免密登录原理：

     ![1571144054503](06_Hadoop_%E8%BF%90%E8%A1%8C%E6%A8%A1%E5%BC%8F/1571144054503.png)

   - 生成公钥和私钥

     在home目录下的.ssh文件夹（隐藏文件夹）下，使用：

     ssh-keygen -t rsa

     然后敲三个回车，就会生成两个文件id_rsa（私钥）、id_rsa.pub（公钥）

   - 将公钥拷贝到要免密登录的目标机器上

     ssh-copy-id hadoop102

     ssh-copy-id hadoop103

     ssh-copy-id hadoop104

     三个都要copy，包括本身；

     在三台机器上都需要生成sshkey，并且三台都要有对方的公钥。

     **注意：在hadoop102上采用root用户，配置无密登录到hadoop102，hadoop103，hadoop104，然后在hadoop103上采用tomxwd用户，配置无密登录到hadoop102，hadoop103，hadoop104上。**

     原因主要是namenode以及resourcemanager需要跟各个服务器互通信息。

   - 除了tomxwd用户外，还需要把root用户也配置ssh

     su root 切换到root用户，重复上一步即可。(如果root目录下没有.ssh文件,就用ssh localhost来生成一次，其实就是用一次ssh即可）

3. .ssh文件夹下（~/.ssh）的文件功能解释

   | 文件名          | 功能解释                                  |
   | --------------- | ----------------------------------------- |
   | known_hosts     | 记录ssh访问过的计算机的公钥（public key） |
   | id_rsa          | 生成的私钥                                |
   | id_rsa.pub      | 生成的公钥                                |
   | authorized_keys | 存放授权过的无密登录服务器公钥            |

   

### 群起集群

1. 配置slaves

   /etc/hadoop/slaves配置文件：

   ```
   hadoop102
   hadoop103
   hadoop104
   ```

   **注意：该文件中添加的内容结尾不允许有空格，文件中不允许有空行。**里面默认的localhost一定要去掉。

   同步所有节点配置文件：

   `xsync slaves`

2. 启动集群

   - hdfs（在hadoop102上启动）

     `start-dfs.sh`

   - yarn（在hadoop103上启动，因为resourcemanager在103上）

     `start-yarn.sh`

     **注意：NameNode和ResourceManager如果不是同一台机器，不能在NameNode上启动YARN，应该在ResourceManager所在的机器上启动YARN**

3. 集群基本测试

   - 上传文件到集群

     - 上传小文件

       `hdfs dfs -mkdir -p /user/tomxwd/input`

       `hdfs dfs -put wcinput/wc.input /user/tomxwd/input`

     - 上传大文件

       `hadoop fs -put /opt/open/download/hadoop-2.9.2.tar.gz /`

   - 上传文件后查看文件存放在什么位置

     - 查看HDFS文件存储路径	

       /opt/open/environment/hadoop-2.9.2/data/tmp/dfs/data/current/BP-918887964-192.168.5.102-1571093054040/current/finalized/subdir0/subdir0

     - 可以把几块文件用>>重定向追加到一个文件中，进行解压，发现是刚刚上传的hadoop压缩包。



### 集群启动/停止方式总结

1. 各个服务组件逐一启动/停止

   - 分别启动/停止HDFS组件

     `hadoop-daemon.sh start/stop namenode/datanode/secondarynamenode`

   - 启动/停止YARN

     `yarn-daemon.sh start/stop resourcemanager/nodemanager`

2. 各个模块分开启动/停止（配置ssh是前提）**常用**

   - 整体启动/停止HDFS
     - start-dfs.sh
     - stop-dfs.sh
   - 整体启动/停止YARN
     - start-yarn.sh
     - stop-yarn.sh



### 集群时间同步（root用户操作）

时间同步的方式：找一个机器，作为时间服务器，所有的机器与这台集群时间进行定时的同步，比如每隔十分钟同步一次时间。

**时间服务器hadoop102----hadoop103定时去获取hadoop102时间服务器主机时间------>其他服务器hadoop103**

- 时间服务器配置：

  1. 检查ntp是否安装

     `rpm -qa | grep ntp`

     ```
     fontpackages-filesystem-1.41-1.1.el6.noarch
     ntp-4.2.6p5-12.el6.centos.2.x86_64
     ntpdate-4.2.6p5-12.el6.centos.2.x86_64
     ```

  2. 修改ntp配置文件

     `vim /etc/ntp.conf`

     - 修改1：授权192.168.1.0-192.168.1.255网段上所有机器可以从这台机器上查询和同步时间

       \# restrict 192.168.1.0 mask 255.255.255.0 nomodify notrap

       改为：

       \# restrict 192.168.1.0 mask 255.255.255.0 nomodify notrap

       这里，192.168.1.0需要改成自己的网段，如192.168.5.0

     - 修改2：集群在局域网中，不使用其他互联网上的时间

       server 0.centos.pool.ntp.org iburst

       server 1.centos.pool.ntp.org iburst

       server 2.centos.pool.ntp.org iburst

       server 3.centos.pool.ntp.org iburst

       改为：

       \# server0 .centos.pool.ntp.org iburst

       \# server1 .centos.pool.ntp.org iburst

       \# server2 .centos.pool.ntp.org iburst

       \# server3 .centos.pool.ntp.org iburst

     - 添加3：当该结点丢失网络连接，依然可以采用本地时间作为时间服务器为集群中的其他结点提供时间同步）

       server 127.127.1.0

       fudge 127.127.1.0 stratum 10

  3. 修改/etc/sysconfig/ntpd文件

     让硬件时间与系统时间一起同步

     `vim /etc/sysconfig/ntpd`

     增加内容：

     ```shell
     SYNC_HWCLOCK=yes
     ```

  4. 重新启动ntpd服务

     `service ntpd status`

     ntpd 已停

     `service ntpd start`

     ntpd 启动

  5. 设置ntpd服务开机启动

     `chkconfig ntpd on`

- 其他服务器配置：

  1. 在其他服务器上配置10分钟与时间服务器同步一次

     `crontab -e`

     内容如下：

     ```shell
     */10 * * * * /usr/sbin/ntpdate hadoop102
     ```

  2. 修改任意机器时间

     `date -s "2019-10-16 21:21:21"`

  3. 十分钟后查看机器是否与时间服务器同步

     `date`

     测试的时候可以调整为1分钟看效果。







## 常见问题

jps发现没进程，但是重启提示已经开启，原因是linux根目录下/tmp目录中存在启动的进程临时文件，将集群相关进程删除，再重新启动集群。