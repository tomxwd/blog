---
2019-08-01 11:36:41

---





默认是使用ConcurrentMapCacheManager创建的ConcurrentMapCache组件，是将数据保存在ConcurrentHashmap中的；

实际在开发中使用的是缓存中间件做缓存的：

- redis
- memcached
- ehcache

SpringBoot支持多种缓存的配置，而默认开启的是SimpleCacheConfiguration。



整合redis作为缓存：

Redis是一个开源（BSD许可）的，内存中的数据结构存储系统，它可以用作数据库、缓存和消息中间件。

1. 安装redis：使用docker;

   `docker run --name myredis -d -p 12388:6379 redis`

2. 可以下载一个redis管理工具来连接尝试是否可用。

   使用docker连接调试：`docker exec -it myredis redis-cli`

3. 可到redis中文网看基本操作。







