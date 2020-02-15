---
title: 03_markword
date: 2019-08-14 16:38:00
tags: 
 - Java
 - 点
categories:
 - 点
---

# 03_markword

markword，java对象头，8个字节，64位，不压缩的情况下。（HotSpot）

头信息包括：

- class p指针：表明这个对象属于哪个类的，指向内存里已经load进来的class字节码。
- 锁信息：锁信息，锁升级信息。
- hashcode：也就是new出来的时候，hashcode已经在markword里面了，所以通过hashcode找对象的时候非常快，不需要再计算。
- GC年代记录：
  - 回收信息，有几个引用指向了这个对象，如果是0了就要被回收。
  - 是收在老年代还是新生代。