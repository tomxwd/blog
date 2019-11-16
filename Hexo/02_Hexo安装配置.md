---
title: 02_Hexo安装配置
date: 2019-11-16 19:16:48
tags: 
 - Hexo
categories:
 - Hexo
---

# 02_Hexo安装配置

[Hexo官网](https://hexo.io/zh-cn/)

直接用` npm install hexo-cli -g `命令即可下载，用`hexo version`命令查看版本。



## 初始化

```shell
hexo init
```

目录结构：

- node_modules：是依赖包
- public：存放的是生成的页面
- scaffolds：命令生成文章等的模板
- source：用命令创建的各种文章
- themes：主题
- _config.yml：整个博客的配置
- db.json：source解析所得到的
- package.json：项目所需模块项目的配置信息



## 创建github仓库

命名格式：

yourname.github.io，只有这个规则才会生效。



## 配置好ssh

```csharp
git config --global user.name “用户名”
git config --global user.email “邮箱”
ssh-keygen -t rsa
```





