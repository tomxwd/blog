---
title: 06_Redis_发布订阅
date: 2019-08-08 15:13:17
tags: 
 - Redis
categories:
 - NoSQL
 - Redis
---

# 06_Redis_发布订阅

## Redis的发布订阅是什么

- 进程间的一种消息通信模式：发送者（pub）发送消息，订阅者（sub）接收消息。

- 订阅/发布消息图

  ![06_redis发布订阅-1](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/06_redis%E5%8F%91%E5%B8%83%E8%AE%A2%E9%98%85-1.png)

## 命令

| 序号 | 命令及描述                                                   |
| :--- | :----------------------------------------------------------- |
| 1    | [PSUBSCRIBE pattern [pattern ...\]](https://www.runoob.com/redis/pub-sub-psubscribe.html)  订阅一个或多个符合给定模式的频道。 |
| 2    | [PUBSUB subcommand [argument [argument ...\]]](https://www.runoob.com/redis/pub-sub-pubsub.html)  查看订阅与发布系统状态。 |
| 3    | [PUBLISH channel message](https://www.runoob.com/redis/pub-sub-publish.html)  将信息发送到指定的频道。 |
| 4    | [PUNSUBSCRIBE [pattern [pattern ...\]]](https://www.runoob.com/redis/pub-sub-punsubscribe.html)  退订所有给定模式的频道。 |
| 5    | [SUBSCRIBE channel [channel ...\]](https://www.runoob.com/redis/pub-sub-subscribe.html)  订阅给定的一个或多个频道的信息。 |
| 6    | [UNSUBSCRIBE [channel [channel ...\]]](https://www.runoob.com/redis/pub-sub-unsubscribe.html)  指退订给定的频道。 |

## 案例

![06_redis发布订阅-2](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/06_redis%E5%8F%91%E5%B8%83%E8%AE%A2%E9%98%85-2.png)