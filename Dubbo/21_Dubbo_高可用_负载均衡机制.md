---
title: 21_Dubbo_高可用_负载均衡机制
date: 2019-09-22 12:17:59
tags: 
 - Dubbo
categories:
 - Java
 - Dubbo
---

# 21_Dubbo\_高可用_负载均衡机制

在集群负载均衡的时候，dubbo提供了多种均衡策略，缺省为random随机调用。



## 负载均衡策略

- Random LoadBalance

  ![1569126710331](21_Dubbo_%E9%AB%98%E5%8F%AF%E7%94%A8_%E8%B4%9F%E8%BD%BD%E5%9D%87%E8%A1%A1%E6%9C%BA%E5%88%B6/1569126710331.png)

  随机，按**权重设置**随机概率

  在一个截面上碰撞的概率高，但调用量越大分布越均匀，而且按概率使用权重后也比较均匀，有利于动态调整提供者权重。

- RoundRobin LoadBalance

  ![1569126741982](21_Dubbo_%E9%AB%98%E5%8F%AF%E7%94%A8_%E8%B4%9F%E8%BD%BD%E5%9D%87%E8%A1%A1%E6%9C%BA%E5%88%B6/1569126741982.png)

  轮询，按公约后的**权重设置**轮询比率。

  存在慢的提供者累积请求的问题，比如：第二台机器很慢，但没挂，当请求调用到第二台时就卡在那里很久，久而久之，所有请求都卡在调用第二台机子上。

- LeastActive LoadBalance

  ![1569126783373](21_Dubbo_%E9%AB%98%E5%8F%AF%E7%94%A8_%E8%B4%9F%E8%BD%BD%E5%9D%87%E8%A1%A1%E6%9C%BA%E5%88%B6/1569126783373.png)

  最少活跃调用数，相同活跃数的随机，活跃数指调用后计数差。

  使慢的提供者收到更少请求，因为越慢的提供者的调用前后计数查会越大。

- ConsistentHash LoadBalance

  ![1569126832888](21_Dubbo_%E9%AB%98%E5%8F%AF%E7%94%A8_%E8%B4%9F%E8%BD%BD%E5%9D%87%E8%A1%A1%E6%9C%BA%E5%88%B6/1569126832888.png)

  一致性哈希，相同参数的请求总是发到同一个提供者。

  当某一台提供者挂掉的时候，原本应该发往该提供者的请求，基于虚拟结点，平摊到其他提供者，不会引起剧烈变动。

  算法参见[网站](http://en.wikipedia.org/wiki/Consistent_hashing)

  缺省只对第一个参数Hash，如果要修改，请配置<dubbo:parameter key="hash.arguments" value="0,1"/>

  缺省用160份虚拟结点，如果要修改，请配置<dubbo:parameter key="hash.nodes" values="320" />

  