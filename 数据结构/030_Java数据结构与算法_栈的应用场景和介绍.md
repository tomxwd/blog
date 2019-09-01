---
title: 030_Java数据结构与算法_栈的应用场景和介绍
date: 2019-07-28 23:04:50
tags: 
 - Java
 - 数据结构与算法
categories:
 - Java
 - 数据结构与算法
---

# 030_Java数据结构与算法_栈的应用场景和介绍

![栈的实际需求](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84/030%E6%A0%88%E7%9A%84%E5%AE%9E%E9%99%85%E9%9C%80%E6%B1%82.png)





![栈的介绍1](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84/030%E6%A0%88%E7%9A%84%E4%BB%8B%E7%BB%8D1.png)

就像玩具枪一样，压子弹，最里面的最后才能发射出来。



![栈的介绍2](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84/030%E6%A0%88%E7%9A%84%E4%BB%8B%E7%BB%8D2.png)





栈的应用场景：

1. 子程序的调用：在跳往子程序前，会先将下一个指令的地址存到堆栈中，直到子程序执行完之后再将地址取出，以回到原来的程序中。return；
2. 处理递归调用：和子程序的调用类似，只是除了储存下一个指令的地址之外，也将参数、区域变量等数据存入堆栈中。
3. 表达式的转换和求值（实际解决）；
4. 二叉树的遍历；
5. 图形的深度优先（depth-first）搜索法；