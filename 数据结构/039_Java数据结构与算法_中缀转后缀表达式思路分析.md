---
title: 039_Java数据结构与算法_中缀转后缀表达式思路分析
date: 2019-08-03 19:08:36
tags: 
 - Java
 - 数据结构与算法
categories:
 - Java
 - 数据结构与算法
---

# 039_Java数据结构与算法_中缀转后缀表达式思路分析

由于逆波兰表达式是对计算机友好，然而对人类并不好阅读，因此我们要进行对中缀表达式到后缀表达式的转换。





![中缀表达式转后缀表达式1](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84/039%E4%B8%AD%E7%BC%80%E8%A1%A8%E8%BE%BE%E5%BC%8F%E8%BD%AC%E5%90%8E%E7%BC%80%E8%A1%A8%E8%BE%BE%E5%BC%8F1.png)![中缀表达式转后缀表达式2](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84/039%E4%B8%AD%E7%BC%80%E8%A1%A8%E8%BE%BE%E5%BC%8F%E8%BD%AC%E5%90%8E%E7%BC%80%E8%A1%A8%E8%BE%BE%E5%BC%8F2.png)![中缀表达式转后缀表达式3](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84/039%E4%B8%AD%E7%BC%80%E8%A1%A8%E8%BE%BE%E5%BC%8F%E8%BD%AC%E5%90%8E%E7%BC%80%E8%A1%A8%E8%BE%BE%E5%BC%8F3.png)

![中缀表达式转后缀表达式4](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84/039%E4%B8%AD%E7%BC%80%E8%A1%A8%E8%BE%BE%E5%BC%8F%E8%BD%AC%E5%90%8E%E7%BC%80%E8%A1%A8%E8%BE%BE%E5%BC%8F4.png)