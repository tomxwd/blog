---
title: 11_SpringCloud_总结
date: 2019-12-11 23:05:41
tags: 
 - SpringCloud
categories:
 - SpringCloud
---

# 11_SpringCloud_总结

1. 整套开发技术栈以SpringCloud为主，单个微服务模块以SpringMVC+SpringBoot/Spring+MyBatis组合进行开发；
2. 前端层，页面H5+thymeleaf/样式CSS3+Boostrap/前端框架JQuery+Node|Vue等；
3. 负载层，前端访问通过Http或Https协议到达服务端的LB，可以是F5等硬件做负载均衡，还可以自行部署LVS+Keepalived等（前期量小可以直接用Nginx）；
4. 网关层，请求通过LB后，会到达整个微服务体系的网管层Zuul（Gateway），内嵌Ribbon做负载均衡，Hystrix做熔断降级等；
5. 服务注册，采用Eureka来做服务治理，Zuul会从Eureka集群获取已发布的微服务访问地址，然后根据配置把请求代理到相应的微服务去；
6. docker容器，所有的微服务模块都部署在Docker容器里面，而且前后端的服务完全分开，各自独立部署前后端微服务调用后端微服务，后端微服务之间会有相互调用；
7. 服务调用，微服务模块间调用都采用标准的Http/Https+REST+JSON的形式，调用技术采用Feign+HttpClient+Ribbon+Hystrix；
8. 统一配置，每个微服务模块会跟Eureka集群、配置中心（SpringCloudConfig）等进行交互；
9. 第三方框架，每个微服务模块根据实现的需要，通常还需要使用一些第三方框架，比如常见的有：缓存服务（redis），图片服务（FastDFS）、搜索服务（ElasticSearch）、安全管理（Shiro）等等；
10. MySQL数据库，可以按照微服务模块进行拆分，统一访问公共库或者单独自己库，可以单独构建MySQL集群或者分库分表MyCat等；



