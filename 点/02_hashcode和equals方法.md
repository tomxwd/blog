---
title: 02_hashcode和equals方法
date: 2019-08-13 15:37:55
tags: 
 - Java
 - 点
categories:
 - 点
---

# 02_hashcode和equals方法

Object.hashCode():

- 记录在markword中
- 由xor-shift算法实现
- 用于哈希表的查找

两个对象对比的时候，先找hashcode取模，看是否为同一个位置，然后再去用equals方法做对比。

所以如果你重写了hashcode，但是没重写equals，还是不能判断是否相同。

因此hashcode和equals一般需要一起重写。

String类就重写了hashcode和equals；

- String重写的hashcode：

- ```
  s[0]*31^(n-1)+s[1]*31^(n-2)+...+s[n-1]
  ```

- 31质数，不大不小的质数，质数做运算的时候容易产生各种各样的值，仅此而已，太大了又容易超出int类型的最大值，这样容易产生重复的值。31刚好是32-1，而32刚好是2的5次方，所以31^n，即是`i<5-i`（位运算，算起来特别快）。

==对比的是两个引用是否相同，equals默认就是==，所以重写了equals，==还是判断引用是否相同，而equals则判断对象是否相同。

好的hash算法：

- 快
- 重复率低（质数的运算重复率就比较低）

