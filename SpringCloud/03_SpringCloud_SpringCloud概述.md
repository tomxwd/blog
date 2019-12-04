---
title: 03_SpringCloud_SpringCloud概述
date: 2019-12-04 20:24:59
tags: 
 - SpringCloud
categories:
 - SpringCloud
---

# 03_SpringCloud_SpringCloud概述

## SpringCloud是什么

**官网说明：**

 ![img](03_SpringCloud_SpringCloud%E6%A6%82%E8%BF%B0/diagram-distributed-systems.svg) 

- SpringCloud，基于Spring提供了一套微服务解决方案，包括服务注册与发现、配置中心、全链路监控、服务网关、负载均衡、熔断器等组件，除了基于NetFlix的开源组件做高度抽象封装之外，还有一些选型中立的开源组件；
- SpringCloud利用SpringBoot的开发便利性巧妙地简化了分布式系统基础设施的开发，SpringCloud为开发人员提供了快速构建分布式系统的一些工具，包括**配置管理、服务发现、断路器、路由、微代理、事件总线、全局锁、决策竞选、分布式会话等等**，他们都可以用SpringBoot的开发风格做到一键启动和部署；
- SpringBoot并没有重复制造轮子，它只是将目前各家公司开发的比较成熟、经得起实际考研的服务框架组合起来，通过SpringBoot风格进行再封装屏蔽掉了复杂的配置和实现原理，**最终给开发者留出了一套简单易懂、易部署和易维护的分布式系统开发工具包**；



**SpringCloud=分布式微服务架构下的一站式解决方案，是各个微服务架构落地技术的集合体，俗称微服务全家桶**



**SpringCloud和SpringBoot是什么关系**

- SpringBoot专注于快速方便的开发单个个体微服务。

- SpringCloud是关注全局的微服务协调整理治理框架，它将SpringBoot开发的一个个单体微服务整合并管理起来，为各个微服务之间提供：**配置管理、服务发现、断路器、路由、微代理、事件总线、全局锁、决策竞选、分布式会话等等**集成服务；

- SpringBoot可以离开SpringCloud独立使用开发项目，但是**SpringCloud离不开SpringBoot**，属于依赖的关系；
- **SpringBoot专注于快速、方便的开发单个微服务个体，SpringCloud关注全局的服务治理框架；**



**Dubbo是怎么到SpringCloud的？哪些优缺点让你去技术选型？**

- 目前成熟的互联网架构（分布式+服务治理Dubbo）

- 我们把SpringCloud和Dubbo进行一番对比

  - 活跃度：

    - [dubbo-github地址](https://github.com/dubbo)
    - [SpringCloud-github地址](https://github.com/spring-cloud)

  - 对比结果：

    |              | Dubbo         | SpringCloud                 |
    | ------------ | ------------- | --------------------------- |
    | 服务注册中心 | Zookeeper     | SpringCloud Netflix Eureka  |
    | 服务调用方式 | RPC           | REST API                    |
    | 服务监控     | Dubbo-monitor | SpringBoot Admin            |
    | 断路器       | 不完善        | SpringCloud Netflix Hystrix |
    | 服务网关     | 无            | SpringCloud Netflix Zuul    |
    | 分布式配置   | 无            | SpringCloud Config          |
    | 服务跟踪     | 无            | SpringCloud Sleuth          |
    | 消息总线     | 无            | SpringCloud Bus             |
    | 数据流       | 无            | SpringCloud Stream          |
    | 批量任务     | 无            | SpringCloud Task            |
    | ......       | ......        | ......                      |

    最大区别：**SpringCloud抛弃了Dubbo的RPC通信，采用的是基于HTTP的REST方式**；

    - 严格来说，这两种方式各有优劣，虽然从一定程度上来说，后者牺牲了服务调用的性能，但也避免了上面提到的原生RPC带来的问题。而且REST相比RPC更为灵活，服务提供方和调用方的依赖只依靠一纸契约，不存在代码级别的强依赖，这在强调快速演化的微服务环境下，显得更加合适。

    - 很明显，SpringCloud的功能比Dubbo更加强大，涵盖面更广，而且作为Spring的拳头项目，它也能够与SpringFramework、SpringBoot、SpringData、SpringBatch等其他的Spring项目完美融合，这些对于微服务而言是至关重要的。使用Dubbo构建的微服务架构就像组装电脑，各个环节我们的选择自由度很高，但是最终结果很有可能因为一条内存质量不行就点不亮了，总是让人不怎么放心，但是如果你是一名高手，那这些都不是问题；而SpringCloud就像品牌机，在SpringSource的整合下，做了大量的兼容性测试，保证了机器拥有更高的稳定性，但是如果要在使用非原装组件之外的东西，就需要对其基础有足够的了解；

    社区支持与更新力度：

    - 最为重要的是，Dubbo停止更新了5年左右，虽然2017.7月重启，但是技术发展的新需求，需要由开发者自行拓展升级（比如当当网弄出了DubboX），这对于很多想要采用微服务架构的中小软件组织，显然是不太适合的，中小公司没有那么强大的技术能力去修改Dubbo源码以及周边的一整套解决方案，并不是每一个公司都有阿里的大牛以及真实的线上生产环境测试过；

- 总结Cloud与Dubbo

  - 问题：曾风靡国内的开源RPC服务框架Dubbo在重启维护后，令许多用户为之雀跃，但是同时，也迎来了一些质疑的声音。互联网技术快速发展，Dubbo是否还能跟上时代？Dubbo与SpringCloud相比又有何优势和差异？是否会有相关举措保证Dubbo的后续更新频率？
  - 人物：Dubbo重启维护开发的刘军，主要负责人之一
    - 刘军，阿里巴巴中间件高级研发工程师，主导了Dubbo重启维护后的几个发行版计划，专注于高性能RPC框架和微服务相关领域。曾负责网易考拉RPC框架的研发以及指导在内部使用，参与了服务治理平台、分布式跟踪系统、分布式一致性框架等从无到有的设计与开发过程；
    - 刘军说：关于Dubbo和SpringCloud的关系，我们在开源中国年终盛典的Dubbo分享中也做了简单的阐述，首先要明确的一点是Dubbo和SpringCloud并不是完全的竞争关系，两者所解决的问题领域不一样：**Dubbo的定位始终是一款RPC框架，而SpringCloud的目标是微服务架构下的一站式解决方案**，如果非要比较的话，我觉得Dubbo可以类比Netflix OSS技术栈，而SpringCloud集成了Netflix OSS作为分布式服务治理解决方案，但除此之外SpringCloud还提供了包括Config、Stream、Security、Sleuth等等分布式问题解决方案；当前由于RPC协议、注册中心元数据不匹配等问题，**在面临微服务基础框架选型的时候，Dubbo与SpringCloud只能二选一**，这也是为什么大家总是拿Dubbo与SpringCloud做对比的原因之一。Dubbo之后会积极寻求匹配到SpringCloud生态，比如作为SpringCloud的二进制通信方案来发挥Dubbo的性能优势，或者Dubbo通过模块化以及对Http的支持适配到SpringCloud；



## SpringCloud能干嘛

- Distributed/versioned configuration（分布式/版本控制配置）
- Service registration and discovery（服务注册与发现）
- Routing（路由）
- Service-toservice calls（服务到服务的调用）
- Load balancing（负载均衡配置）
- Circuit Breakers（断路器）
- Distributed messaging（分布式消息管理）
- ......





## SpringCloud去哪里下载

[官网](https://spring.io/projects/spring-cloud)

参考：

- [Spring Cloud Netflix中文文档](https://springcloud.cc/spring-cloud-netflix.html)
- [SpringCloud-Dalston中文文档](https://springcloud.cc/spring-cloud-dalston.html)
- [SpringCloud中文社区](https://springcloud.cn)
- [SpringCloud中文网](https://springcloud.cc)

## SpringCloud怎么玩

- 服务的注册与发现（Eureka）
- 服务消费者（REST+Ribbon）
- 服务消费者（Feign）
- 断路器（Hystrix）
- 断路器监控（Hystrix Dashboard）
- 路由网关（Zuul）
- 分布式配置中心（SpringCloud Config）
- 消息总线（SpringCloud Bus）
- 服务链路追踪（SpringCloud Sleuth）
- ......







