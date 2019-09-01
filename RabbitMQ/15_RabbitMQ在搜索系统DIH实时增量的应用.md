---
title: 15_RabbitMQ在搜索系统DIH实时增量的应用
date: 2019-07-01 22:13:42
tags: 
 - Java
 - RabbitMQ
categories:
 - MQ
 - RabbitMQ
---

# 15_RabbitMQ在搜索系统DIH实时增量的应用

DIH（Data Import Handler）

比如在Solr索引库中的应用，数据库增删改数据的时候，需要同步索引库，以往可能通过定时任务来解决这个问题，但是可能会导致消息不同步，可以改为用MQ来解决。

