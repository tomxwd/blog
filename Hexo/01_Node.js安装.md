---
title: 01_Node.js安装
date: 2019-11-16 18:44:29
tags: 
 - Hexo
categories:
 - Hexo
---

# 01_Node.js安装

## 安装

[node.js官网](https://nodejs.org/en/)

下载LTS版本即可。



安装步骤除了换安装路径之外，其他一路确定即可。



最后在命令行窗口用node -v看node版本，以及npm -v查看npm版本即可。



创建两个文件夹用于保存npm产生的文件，避免造成系统盘内存使用。

```shell
npm config set prefix "E:\soft\node_global"

npm config set cache "E:\soft\node_cache"
```



创建环境变量NODE_PATH，值为E:\open\soft\node.js\node_modules，也就是Node.js里面的node_modules文件夹。

![image-20191116185931489](01_Node.js%E5%AE%89%E8%A3%85/image-20191116185931489.png)



最后编辑用户变量里的Path，将npm的路径改为node_global：

![image-20191116190247130](01_Node.js%E5%AE%89%E8%A3%85/image-20191116190247130.png)



测试一下：

执行

```shell
npm install hexo-cli -g
```

顺便安装了hexo-cli，然后到node_global以及node_cache文件查看是否生效。

然后执行

```shell
hexo version
```

查看hexo版本



## 切换仓库源

用`npm config ls`查看npm信息

![image-20191116190957535](01_Node.js%E5%AE%89%E8%A3%85/image-20191116190957535.png)

- metrics-registry：仓库源
- prefix：本地仓库地址，需要和环境变量Path里的路径对上。

执行命令切换淘宝的仓库源：

```shell
npm config set registry https://registry.npm.taobao.org/
```

需要切换回去则执行：

```shell
npm config set registry https://registry.npmjs.org
```

更换本地仓库地址以及缓存地址：

```shell
npm config set prefix "E:\soft\node_global"

npm config set cache "E:\soft\node_cache"
```

需要配置环境变量。

