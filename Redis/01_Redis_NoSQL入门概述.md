---
title: 01_Redis_NoSQL入门概述
date: 2019-08-06 09:02:06
tags: 
 - Redis
categories:
 - NoSQL
 - Redis
---

# 01_Redis_NoSQL入门概述

## 入门概述

### 1、互联网时代背景下大机遇，为什么用nosql

1. 单机MySQL的美好年代

   在90年代,一个网站的访问量并不大，用单个数据库完全可以轻松应付。

   在那个时候，更多的都是静态网页，动态交互类型的网站不多。

   ![01_NoSql入门概述-1](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/01_NoSql%E5%85%A5%E9%97%A8%E6%A6%82%E8%BF%B0-1.png)

   上述架构下，我们来看看数据存储的瓶颈是什么？

   - 数据量的总大小，一个机器放不下的时候。
   - 数据的索引（B+Tree），一个机器的内存放不下的时候。
   - 访问量（读写混合），一个实例不能承受的时候。

2. Memcached(缓存)+MySQL+垂直拆分

   ![01_NoSql入门概述-2](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/01_NoSql%E5%85%A5%E9%97%A8%E6%A6%82%E8%BF%B0-2.png)

3. MySQL主从读写分离

   ![01_NoSql入门概述-3](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/01_NoSql%E5%85%A5%E9%97%A8%E6%A6%82%E8%BF%B0-3.png)

4. 分表分库+水平拆分+MySQL集群

   ![01_NoSql入门概述-4](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/01_NoSql%E5%85%A5%E9%97%A8%E6%A6%82%E8%BF%B0-4.png)

   ![01_NoSql入门概述-5](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/01_NoSql%E5%85%A5%E9%97%A8%E6%A6%82%E8%BF%B0-5.png)

5. MySQL的扩展性瓶颈

   ![01_NoSql入门概述-6](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/01_NoSql%E5%85%A5%E9%97%A8%E6%A6%82%E8%BF%B0-6.png)

6. 今天是什么样子??

   ![01_NoSql入门概述-7](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/01_NoSql%E5%85%A5%E9%97%A8%E6%A6%82%E8%BF%B0-7.png)

   ![01_NoSql入门概述-8](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/01_NoSql%E5%85%A5%E9%97%A8%E6%A6%82%E8%BF%B0-8.png)

7. 为什么用NoSQL

   ![01_NoSql入门概述-9](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/01_NoSql%E5%85%A5%E9%97%A8%E6%A6%82%E8%BF%B0-9.png)

### 2、是什么

![01_NoSql入门概述-10](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/01_NoSql%E5%85%A5%E9%97%A8%E6%A6%82%E8%BF%B0-10.png)

### 3、能干嘛

1. 易扩展

   NoSQL数据库种类繁多，但是一个共同的特点都是去掉关系数据库的关系型特性。

   数据之间无关系，这样就非常容易扩展，也无形之间在架构的层面上带来了可扩展的能力。

2. 大数据量高性能

   NoSQL数据库都具有非常高的读写性能，尤其在大数据量下，同样表现优秀，这得益于它的无关系性，数据库的结构简单。

   一般MySQL使用Query Cache，每次表的更新Cache就失效，是一种大粒度的Cache，在针对web2.0的交互频繁的应用，Cache性能不高。而NoSQL的Cache是记录级的，是一种细粒度的Cache，所以NoSQL在这个层面上来说就要性能高很多了。

3. 多样灵活的数据模型

   NoSQL无需事先为要存储的数据建立字段，随时可以存储自定义的数据格式。而在关系型数据库里，增删字段是一件非常麻烦的事前，如果是非常大数据量的表，增加字段简直就是一个噩梦；

4. 传统RDBMS VS NOSQL

   - RDBMS
     - 高度组织化结构化数据
     - 结构化查询语句（SQL）
     - 数据和关系都存储在单独的表中
     - 数据操纵语言，数据定义语言
     - 严格的一致性
     - 基础事务
   - NoSQL
     - 代表着不仅仅是SQL
     - 没有声明性查询语句
     - 没有预定义的模式
     - 键值对存储，列存储，文档存储，图形数据库
     - 最终一致性，而非ACID属性
     - 非结构化和不可预知的数据
     - CAP定理
     - 高性能、高可用、可伸缩



### 4、去哪下

1. Redis
2. Memcache
3. Mongdb

### 5、怎么玩

1. KV
2. Cache
3. Persistence
4. ......



## 3V+3高

### 1、大数据时代的3V

1. 海量Volume
2. 多样Variety
3. 实时Velocity

### 2、互联网需求的3高

1. 高并发
2. 高可扩
3. 高性能



## 当下的NoSQL经典应用

当下的应用是sql和NoSQL一起使用

### 阿里巴巴中文站商品信息如何存放

#### 看看阿里巴巴中文网站首页，以女装/女包包为例

![01_NoSql入门概述-11](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/01_NoSql%E5%85%A5%E9%97%A8%E6%A6%82%E8%BF%B0-11.png)

##### 架构发展历程

1. 演变历程

   ![01_NoSql入门概述-12](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/01_NoSql%E5%85%A5%E9%97%A8%E6%A6%82%E8%BF%B0-12.png)

   ![01_NoSql入门概述-13](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/01_NoSql%E5%85%A5%E9%97%A8%E6%A6%82%E8%BF%B0-13.png)

2. 第5代

   ![01_NoSql入门概述-14](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/01_NoSql%E5%85%A5%E9%97%A8%E6%A6%82%E8%BF%B0-14.png)

3. 第5代架构使命

   ![01_NoSql入门概述-15](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/01_NoSql%E5%85%A5%E9%97%A8%E6%A6%82%E8%BF%B0-15.png)

4. ......

##### 和我们相关的，多数据源多数据类型的存储问题

![01_NoSql入门概述-16](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/01_NoSql%E5%85%A5%E9%97%A8%E6%A6%82%E8%BF%B0-16.png)

![01_NoSql入门概述-17](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/01_NoSql%E5%85%A5%E9%97%A8%E6%A6%82%E8%BF%B0-17.png)

#### 淘宝商品

1. 商品基本信息

   名称、价格、出厂日期、生产厂商等。

   关系型数据库，MySQL/Oracle目前淘宝在去O化（也就是拿掉Oracle），**注意，淘宝内部用的MySQL是里面的大牛自己改造过的**；

   - 为什么去IOE

     ![01_NoSql入门概述-18](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/01_NoSql%E5%85%A5%E9%97%A8%E6%A6%82%E8%BF%B0-18.png)

2. 商品描述、详情、评价信息（多文字类）

   多文字信息描述类，IO读写性能变差，存在文档数据库MongDB中。

3. 商品的图片

   商品图片展示类，在分布式的文件系统中：

   - 淘宝自己的TFS
   - Google的GFS
   - Hadoop的HDFS

4. 商品的关键字

   搜索引擎，淘宝内部使用ISearch

5. 商品的波段性的热点高频信息

   内存数据库：Tair，Redis、Memcache

6. 商品的交易、价格计算、积分累计

   外部系统，外部第三方支付接口，支付宝

7. 总结大型互联网应用（大数据、高并发、多样数据类型）的难点和解决方案

   - 难点

     - 数据类型的多样性
     - 数据源多样性和变化重构
     - 数据源改造而数据服务平台不需要大面积重构

   - 解决办法

     EAI和统一数据平台服务层，阿里淘宝UDSL

     类似面向接口编程，提供一个接口去操作多个数据源，统一数据平台服务层。

     **UDSL：**

     - 是什么

       ![01_NoSql入门概述-19](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/01_NoSql%E5%85%A5%E9%97%A8%E6%A6%82%E8%BF%B0-19.png)

       ![01_NoSql入门概述-20](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/01_NoSql%E5%85%A5%E9%97%A8%E6%A6%82%E8%BF%B0-20.png)

     - 什么样

       ![01_NoSql入门概述-21](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/01_NoSql%E5%85%A5%E9%97%A8%E6%A6%82%E8%BF%B0-21.png)

       1. 映射

          ![01_NoSql入门概述-22](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/01_NoSql%E5%85%A5%E9%97%A8%E6%A6%82%E8%BF%B0-22.png)

          ![01_NoSql入门概述-23](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/01_NoSql%E5%85%A5%E9%97%A8%E6%A6%82%E8%BF%B0-23.png)

       2. API

          ![01_NoSql入门概述-24](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/01_NoSql%E5%85%A5%E9%97%A8%E6%A6%82%E8%BF%B0-24.png)

          ![01_NoSql入门概述-25](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/01_NoSql%E5%85%A5%E9%97%A8%E6%A6%82%E8%BF%B0-25.png)

       3. 热点缓存

          ![01_NoSql入门概述-26](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/01_NoSql%E5%85%A5%E9%97%A8%E6%A6%82%E8%BF%B0-26.png)

          ![01_NoSql入门概述-27](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/01_NoSql%E5%85%A5%E9%97%A8%E6%A6%82%E8%BF%B0-27.png)

       4. ......



## NoSQL数据模型简介

### 以一个电商客户、订单、订购、地址模型来对比下关系型数据库和非关系型数据库

1. 传统的关系型数据库你如何设计？

   - E-R图（1:1/1:N/N:N，主外键等）

     ![01_NoSql入门概述-28](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/01_NoSql%E5%85%A5%E9%97%A8%E6%A6%82%E8%BF%B0-28.png)

   - NoSQL如何设计

     - 什么是BSON

       BSON()是一种类JSON的一种二进制形式的存储格式，简称Binary JSON，和JSON一样支持内嵌的文档对象和数组对象。

       ![01_NoSql入门概述-29](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/01_NoSql%E5%85%A5%E9%97%A8%E6%A6%82%E8%BF%B0-29.png)

     - 两者对比，问题和难点

       - 为什么上述的情况可以用聚合模型来处理

         高并发的操作是不太建议有关联查询的，互联网公司用冗余数据来避免关联查询，分布式事务是支持不了太多的并发的。

       - 关系模型数据库如何查？按照BSON查询

### 聚合模型

1. KV键值对

2. BSON

3. 列族

   顾名思义，是按列存储数据的。最大的特点是方便存储结构化和半结构化数据，方便做数据压缩，对针对某一列或者某几列的查询有非常大的IO优势。

   ![01_NoSql入门概述-30](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/01_NoSql%E5%85%A5%E9%97%A8%E6%A6%82%E8%BF%B0-30.png)

4. 图族

   ![01_NoSql入门概述-31](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/01_NoSql%E5%85%A5%E9%97%A8%E6%A6%82%E8%BF%B0-31.png)

   ![01_NoSql入门概述-32](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/01_NoSql%E5%85%A5%E9%97%A8%E6%A6%82%E8%BF%B0-32.png)



## NoSQL数据库的四大分类

### KV键值：典型介绍

1. 新浪：BerkeleyDB+redis
2. 美团：redis+tair
3. 阿里、百度：memcache+redis

### 文档型数据库（bson格式比较多）：典型介绍

1. CouchDB

2. MongoDB

   MongoDB是一个**基于分布式文件存储的数据库**，由C++语言编写，旨在为WEB应用提供可扩展的高性能数据存储解决方案。

   MongoDB是一个介于关系数据库和非关系数据库之间的产品，是非关系型数据库当中功能最丰富，最像关系型数据库的。

### 列存储数据库

Cassandra，HBase，分布式文件系统

### 图关系数据库

不是放图形的，而是放关系，比如：朋友圈社交网络、广告推荐系统、社交网络推荐系统等，专注于构建关系图谱，如Neo4J，InfoGrid

### 四者对比

![01_NoSql入门概述-33](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/01_NoSql%E5%85%A5%E9%97%A8%E6%A6%82%E8%BF%B0-33.png)

![01_NoSql入门概述-34](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/01_NoSql%E5%85%A5%E9%97%A8%E6%A6%82%E8%BF%B0-34.png)



## 在分布式数据库中CAP原理CAP+BASE

### 传统的ACID分别是什么

![01_NoSql入门概述-35](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/01_NoSql%E5%85%A5%E9%97%A8%E6%A6%82%E8%BF%B0-35.png)

![01_NoSql入门概述-36](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/01_NoSql%E5%85%A5%E9%97%A8%E6%A6%82%E8%BF%B0-36.png)

### CAP

![01_NoSql入门概述-37](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/01_NoSql%E5%85%A5%E9%97%A8%E6%A6%82%E8%BF%B0-37.png)

### CAP的3进2

![01_NoSql入门概述-38](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/01_NoSql%E5%85%A5%E9%97%A8%E6%A6%82%E8%BF%B0-38.png)

![01_NoSql入门概述-39](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/01_NoSql%E5%85%A5%E9%97%A8%E6%A6%82%E8%BF%B0-39.png)

![01_NoSql入门概述-40](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/01_NoSql%E5%85%A5%E9%97%A8%E6%A6%82%E8%BF%B0-40.png)

![01_NoSql入门概述-41](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/01_NoSql%E5%85%A5%E9%97%A8%E6%A6%82%E8%BF%B0-41.png)

### 经典CAP图

CAP理论的核心是：一个分布式系统不可能同时很好的满足一致性、可用性和分区容错性这三个需求，**最多只能同时较好的满足两个；**

因此，根据CAP原理将NoSQL数据库分成了满足CA原则、满足CP原则和满足AP原则三大类：

- CA：单点集群，满足一致性、可用性的系统，通常在可扩展性上不太强大；
- CP：满足一致性，分区容忍性的系统，通常性能不是特别高；
- AP：满足可用性，分区容忍性的系统，通常可能对一致性要求低一点；

![01_NoSql入门概述-42](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/01_NoSql%E5%85%A5%E9%97%A8%E6%A6%82%E8%BF%B0-42.png)

### BASE

![01_NoSql入门概述-43](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/01_NoSql%E5%85%A5%E9%97%A8%E6%A6%82%E8%BF%B0-43.png)

### 分布式+集群简介

分布式系统：

由多台计算机和通信的软件组件通过计算机网络连接（本地网络或广域网）组成。分布式系统是建立在网络之上的软件系统，正式因为软件的特性，所以分布式系统具有高度的内聚性和透明性。因此，网络和分布式系统之间的区别更多在于高层软件（特别是操作系统），而不是硬件。分布式系统可以应用在不同的平台上：PC、工作站、局域网和广域网上等。

简单来讲：

1. 分布式：不同的多台服务器上面部署不同的服务模块（工程），他们之间通过RPC/RMI之间通信和调用，对外提供服务和组内协作；
2. 集群：不同的多台服务器上面部署相同的服务模块，通过分布式调度软件进行统一的调度，对外提供服务和访问；