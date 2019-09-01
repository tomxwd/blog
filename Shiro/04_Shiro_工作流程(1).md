---
title: 04_Shiro_工作流程(1)
date: 2019-08-29 15:39:57
tags: 
 - Java
 - Shiro
categories:
 - Shiro
---

# 04_Shiro_工作流程(1)

## 与Web集成

- Shiro提供了与Web集成的支持，其通过一个**ShiroFilter**入口来拦截需要安全控制的URL，然后进行相应的控制；
- ShiroFilter类似于如Strut2/SpringMvc这种web框架的前端控制器，是**安全控制的入口点**，其负责读取配置（如ini配置文件），然后**判断URL是否需要登录/权限等工作**；



## ShiroFIlter工作原理

![ShiroFIlter工作原理](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/Shiro/04ShiroFIlter%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86.png)

