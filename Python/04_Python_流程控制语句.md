---
title: 04_Python_流程控制语句
date: 2019-09-25 21:30:34
tags: 
 - Python
categories:
 - Python
---

# 04_Python_流程控制语句

## 条件判断语句（if语句）

语法：

1. if 条件表达式：语句

2. if条件表达式：

   ​	代码块



执行的流程：if语句在执行的时候，先对条件表达式进行求值判断：

- 如果为True则执行if后的语句
- 如果为False则不执行

默认情况下，if语句只会控制紧随其后的一条语句，如果希望if可以控制多条语句，则可以在if后跟着一个代码块：

如果要编写代码块，语句就不能紧随其后写，而是写在下一行

```python
if True :
	print("123")
	print("456")
```

代码块以缩进来开始，直到代码恢复到之前的缩进级别的时候结束



## 条件判断语句（if-else语句）

语法：

if 条件表达式：

​	代码块

else：

​	代码块



## 条件判断语句（if-elif-else语句）

语法：

if 条件表达式：

​	代码块

elif：

​	代码块

elif：

​	代码块

else：

​	代码块



一个条件满足了，就只执行这个代码块

