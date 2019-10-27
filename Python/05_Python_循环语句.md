---
title: 05_Python_循环语句
date: 2019-09-26 20:22:04
tags: 
 - Python
categories:
 - Python
---

# 05_Python_循环语句

循环语句可以使指定的代码块重复指定的次数

循环语句分成两种，while循环和for循环



## while循环

语法1：

while 条件表达式：

​	代码块



语法2：

while 条件表达式：

​	代码块

else：

​	代码块



执行流程：

1. 判断条件表达式的值，如果为True则执行，直到False
2. 如果有else代码块，则执行一次else代码块的内容



## break和continue

break可以用来立即退出循环语句（包括else）

continue不会对else产生影响



## while练习

### 水仙花数练习

求1000以内个位十位百位的立方和为该数：

```python
i = 100
while i < 1000:
	a = i//100
	b = ( i - 100 * a )// 10
	c = i - 100 * a - 10 * b
	if a ** 3 + b ** 3 + c ** 3 == i:
		print(i)
	i += 1
```

结果：

```
153
370
371
407
```



### 质数练习

100以内的质数：

```python
i = 1
sum = 0
while i < 100:
	j = 2
	while j < i:
		if i % j == 0:
			sum += 1
			print(i,j)
			break
		j += 1
	i += 1
```


