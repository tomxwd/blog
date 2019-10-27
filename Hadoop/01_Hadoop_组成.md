---
title: 01_Hadoop_组成
date: 2019-09-29 20:37:46
tags: 
 - Hadoop
categories:
 - Hadoop
---

# 01_Hadoop_组成

![1569762325816](01_Hadoop_Hadoop%E7%BB%84%E6%88%90/1569762325816.png)

## HDFS架构概述

HDFS(Hadoop Distributed File System)的架构概述

1. NameNode(nn)：存储文件的元数据，如文件名，文件目录结构，文件属性（生成时间、副本数、文件权限），以及每个文件的块列表和块所在的DataNode等。
2. DataNode(dn)：在本地文件系统存储文件块数据，以及块数据的校验和。
3. Secondary NameNode(2nn)：用来监控HDFS状态的辅助后台程序，每隔一段时间获取HDFS元数据的快照。

![1569762047162](01_Hadoop_Hadoop%E7%BB%84%E6%88%90/1569762047162.png)

## MapReduce架构概述

MapReduce将计算过程分为两个阶段：Map和Reduce

1. Map阶段并行处理输入数据
2. Reduce阶段对Map结果进行汇总

![1569762268963](01_Hadoop_Hadoop%E7%BB%84%E6%88%90/1569762268963.png)