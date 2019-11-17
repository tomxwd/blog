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



## 配置hexo

全局_config.yml文件中增加配置：

```yaml
deploy:
  type: git
  repo: git@github.com:tomxwd/tomxwd.github.io.git
  branch: master
```



然后执行：

```shell
hexo clean
hexo generate
hexo server
```

安装hexo-cli的可以执行hexo server，而直接安装hexo的则可能需要单独安装hexo-server；

hexo3.0之后，需要单独安装hexo-server，执行`npm i hexo-server`才可以执行hexo server命令，如果`hexo server`可以执行，就不用装了。



## 上传github

需要安装插件：

```sh
npm install hexo-deployer-git --save
```

然后执行命令：

```shell
hexo clean
hexo generate
hexo deploy
```



## 绑定个人域名

在source文件夹下新建一个名为CNAME的文件，在里面加入域名blog.tomxwd.top

再次部署即可。