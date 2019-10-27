---
title: 01_Python_环境搭建
date: 2019-09-22 23:14:19
tags: 
 - Python
categories:
 - Python
---

# 01_Python_环境搭建

开发环境搭建就是安装Python的解释器；

Python的解释器分类：

- CPython：用C语言编写的Python解释器【官方】
- PyPy：用Python语言编写的Python解释器
- IronPython：用.net编写的Python解释器
- Jython：用Java编写的Python解释器



## 安装步骤

1. 下载安装包，[官网](https://www.python.org/)
2. 安装（傻瓜式安装）
   - 选择自定义安装，因为他的默认路径太难找了，而且在C盘。
   - 然后Add Python to environment variables打钩，即把Python放到环境变量path中去。
   - 可能会提示path超出限制，点解除就好了。
3. 打开命令行窗口，输入Python检查是否安装成功；这里会进入到Python的交互界面；



## 交互界面

输入python之后，会出现机器上Python的版本等信息。

命令提示符：\>\>\>

在命令提示符后面可以直接输入Python的指令，输入完的指令将会被Python的解释器立即执行；



Python安装完后会有一个IDLE，通过这个IDLE也可以进入到交互模式；不同的是在IDLE中可以用tab键来查看语句的提示，并且可以把代码保存。



交互模式只能输入一行，执行一行，所以并不适用于日常开发；仅可以用来做一些日常的简单测试。



我们一般将Python代码编写到一个py文件中，然后通过Python指令来执行文件中的代码；