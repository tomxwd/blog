---
title: 02_Node.js_包简介
date: 2019-11-23 09:18:17
tags: 
 - node.js
categories:
 - node.js
---

## 02_Node.js_包简介

- CommonJS的包规范允许我们将一组相关的模块组合在一起，形成一组完整的工具。
- CommonJS的包规范由**包结构**和**宝描述文件**两个部分组成
  - 包结构
    - 用于组织包中的各种文件
    - 包实际上就是一个压缩文件，解压以后还原为目录。符合规范的目录，应该包含如下文件：
      - **package.json	描述文件（必须要有）**
      - bin		              可执行二进制文件
      - lib						js代码
      - doc					 文档
      - test					 单元测试
  - 包描述文件
    - 描述包的相关信息，以供外部读取分析



## NPM(Node Package Manager)

- CommonJS包规范是理论，NPM是其中的一种实践。
- 对于Node而言，NPM帮助其完成了第三方模块的发布、安装和依赖等。借助NPM，Node与第三方模块之间形成了很好的一个生态系统。



### NPM命令

- npm -v
  - 查看版本
- npm
  - 帮助说明
- npm search 包名
  - 搜索模块包
- npm install / i 包名
  - 在当前目录安装包
- **npm install 包名 -g**
  - **全局模式安装包**
  - 一般都是一些工具
- **npm init**
  - **初始化，可用于创建package.json文件**
- npm remove 包名
  - 删除一个模块
- **npm install 文件路径**
  - **从本地安装**
  - 会找依赖去下载，形成node_module文件夹
- npm install 包名 -registry=地址
  - 从镜像源安装
- npm config set registry 地址
  - 设置镜像源
- **npm install 包名 --save**
  - **安装包并添加到依赖中**



### 配置cnpm

npm默认是从外网拉取包，所以在国内改成淘宝镜像。

[淘宝npm镜像官网](https://npm.taobao.org/)

执行：

```shell
npm install -g cnpm --registry=https://registry.npm.taobao.org
```

然后以后想用国内镜像进行下载，就使用cnpm指令。



## node搜索包流程

- node在使用模块名字来引入模块时，会首先在当前目录的node_modules中寻找是否含有该模块：
  - 如果有则直接使用。
  - 如果没有则去上一级目录的node_modules中寻找
    - 如果有则直接使用，则再去上一级寻找，直到找到为止。
- 直到找到磁盘的根目录，如果依然没有则报错。