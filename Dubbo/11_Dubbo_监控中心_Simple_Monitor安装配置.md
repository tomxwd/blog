---
title: 11_Dubbo_监控中心_Simple_Monitor安装配置
date: 2019-09-11 21:29:46
tags: 
 - Dubbo
categories:
 - Java
 - Dubbo
---

# 11_Dubbo\_监控中心_Simple_Monitor安装配置

## 安装监控中心

> 仍然在[dubbo-admin这里](https://github.com/apache/dubbo-admin/tree/master)下载

在incubator-dubbo-ops-master文件夹中有个dubbo-monitor-simple。

1. 改dubbo-monitor-simple中的配置文件，如zookeeper端口等。
2. 对dubbo-monitor-simple进行打包
3. 打开target目录发现有jar包以及有个压缩包【dubbo-monitor-simple-2.0.0-assembly.tar.gz】，复制这个压缩包出来，并解压
4. 运行assembly.bin中的start.bat启动监控中心
5. 看到控制台输出Dubbo service server started!即可到浏览器访问localhost:8080



## 在项目中配置监控中心信息

要在项目配置文件中对监控中心的信息进行配置，监控中心才会监控到服务等信息。

其实只要配一个dubbo:monitor标签，两个属性：

- protocol：缺省值为dubbo，默认是直连监控中心，如果指定为registry，则会去监控中心找，这时候不需要配地址。
- address：配置监控中心地址，直连配置。

消费者跟生产者都要进行这个配置：

```xml
<dubbo:monitor protocol="registry"></dubbo:monitor>
<!--<dubbo:monitor address="localhost:7070"></dubbo:monitor>-->
```

**注意**：7070是通信端口，在monitor配置文件可以看到。