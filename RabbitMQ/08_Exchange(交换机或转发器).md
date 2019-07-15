---
title: 08_Exchange(交换机/转发器)
data: 2019-06-22 ‏‎‏‎‏‎21:55:24
tags: 
 - Java
 - RabbitMQ
categories:
 - MQ
 - RabbitMQ
---

# 08_Exchange(交换机/转发器)

一方面是接收生产者的消息，另一方面是向队列推送信息。



匿名转发 “”（为空）



---

## fanout（不处理路由键）

![fanout](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/RabbitMQ/08fanout.png)

只需要简单的将队列绑定到交换机上。

发送到交换机的消息都会被转发到与该交换机绑定的所有队列上。

**Fanout交换机转发消息是最快的。**



---

## Direct（处理路由键）

![direct](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/RabbitMQ/08direct.png)

所有发送到Direct Exchange的消息被转发RoutingKey中指定的Queue

**注意：**Direct模式可以使用RabbitMQ自带的Exchange:default Exchange，所以不需要将Exchange进行任何的绑定（binding）操作，消息传递时，RoutingKey必须完全匹配才会被队列接受，否则该信息会被抛弃。



---

## Topic（某模式进行匹配）

![direct](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/RabbitMQ/08topic.png)

将路由键和某模式匹配

- \# 匹配一个或者多个词
- \* 只会匹配一个词

Goods.# 可以拿到Goods.开头的所有

