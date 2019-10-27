---
title: 02_SublimeText_与Python整合
date: 2019-09-23 19:55:20
tags: 
 - SublimeText
categories:
 - SublimeText
---

# 02_SublimeText_与Python整合

1. 在Sublime中执行Python代码，Ctrl+B自动在Sublime内置的控制台中执行；

   这种执行方式，在某些版本的Sublime中对中文支持不好，并且不能使用input()函数

2. 使用SublimeREPL来运行Python代码

   - Package Control - > install package - > SublimeREPL

   - 安装完毕，到工具 - > SublimeREPL - > Python

     - Python：交互界面
     - Python - RUN current file：运行当前文件

   - 希望按F5就自动执行当前的Python代码

     ```json
     [
         {
             "keys": ["F5"],
             "caption": "SublimeREPL:Python",
             "command": "run_existing_window_command",
             "args": {
                 "id": "repl_python_run",
                 "file": "config/Python/Main.sublime-menu"
             }
         },
     ]
     ```

     找到首选项 -> 快捷键设置，然后把内容复制进去即可。

3. 加一个线来标识多少个字符

   在首选项 - > 设置 - > 添加

   ```json
   "rulers":[
       80
   ],
   ```

   

4. 