---
title: 03_Redis_解析配置文件
date: 2019-08-07 15:06:48
tags: 
 - Redis
categories:
 - NoSQL
 - Redis
---

# 03_Redis_解析配置文件

## 1. 配置文件的位置

### 地址

在redis解压目录下就有一个redis.conf文件，最好拷贝出来使用。

## 为什么要拷贝出来使用

并不能保证一次就能修改正确，且可能需要配置不一样的情况。

## 2. Units单位

![03_redis配置文件-1](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/03_redis%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6-1.png)

## 3. INCLUDES包含

![03_redis配置文件-2](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/03_redis%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6-2.png)

## 4. GENERAL通用

![03_redis配置文件-3](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/03_redis%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6-3.png)

- Daemonize

- Pidfile

- Port

- Tcp-backlog

  ![03_redis配置文件-4](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/03_redis%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6-4.png)

- Timeout

- Bind

- Tcp-keepalive

  ![03_redis配置文件-5](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/03_redis%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6-5.png)

- Loglevel

- Logfile

- Syslog-enabled

  是否把日志输出到syslog中

- Syslog-ident

  指定syslog里的日志标志

- Syslog-facility

  指定syslog设备，值可以是USER或LOCAL0-LOCAL7

- Databases

  库的个数，默认16个库

## 5. SNAPSHOTTING快照

1. save

   - save秒钟，写操作次数

     ![03_redis配置文件-6](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/03_redis%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6-6.png)

   - 禁用

     ![03_redis配置文件-7](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/03_redis%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6-7.png)

2. stop-writes-on-bgsave-error

   ![03_redis配置文件-8](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/03_redis%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6-8.png)

3. rdbcompression

   ![03_redis配置文件-9](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/03_redis%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6-9.png)

4. rdbchecksum

   ![03_redis配置文件-10](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/03_redis%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6-10.png)

5. dbfilename

   文件名

6. dir

   目录

## 6. REPLICATION复制



## 7. SECURITY安全

访问密码的查看、设置和取消；

redis-cli连接上后，可以通过`config get requirepass`查看密码，也可以用`config set requirepass "密码"`来设定密码，这时候再用`ping`则会弹出需要你输入密码的提示。这时候用`auto 密码`即可再次`ping`通。

## 8. LIMITS限制

高版本为CLIENTS等。

### CLIENTS

- maxclients 10000

### MEMORY MANAGEMENT

- maxmemory \<bytes\>

- maxmemory-policy noeviction

  超过最大内存【maxmemory】的时候，使用过期策略。

  ```shell
  # volatile-lru -> Evict using approximated LRU among the keys with an expire set.
  # allkeys-lru -> Evict any key using approximated LRU.
  # volatile-lfu -> Evict using approximated LFU among the keys with an expire set.
  # allkeys-lfu -> Evict any key using approximated LFU.
  # volatile-random -> Remove a random key among the ones with an expire set.
  # allkeys-random -> Remove a random key, any key.
  # volatile-ttl -> Remove the key with the nearest expire time (minor TTL)
  # noeviction -> Don't evict anything, just return an error on write operations.
  ```

  ![03_redis配置文件-11](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/03_redis%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6-11.png)

  - noeviction：永不过期

- maxmemory-samples 5

  ![03_redis配置文件-12](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/03_redis%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6-12.png)

- replica-ignore-maxmemory yes

## 9. APPEND ONLY MODE追加

- appendonly yes/no ：是否开启aof配置

- appendfilename 文件名

- appendfsync

  同步策略：

  - always：同步持久化，每次发生数据变更都会被立即记录到磁盘，性能较差，但是数据完整性比较好；
  - everysec：**出厂默认推荐**，异步操作，每秒记录，如果一秒内宕机，会有数据丢失；
  - No：从不；

  ![03_redis配置文件-13](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/03_redis%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6-13.png)

- no-appendfsync-on-rewrite：重写时是否可以运用appendfsync，用默认no即可，保证数据安全性。

- auto-aof-rewrite-min-size：设置重写的基准值，

- auto-aof-rewrite-percentage：设置重写的基准值，

## 10. 常见配置redis.conf介绍

![03_redis配置文件-14](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/03_redis%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6-14.png)

![03_redis配置文件-15](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/03_redis%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6-15.png)

![03_redis配置文件-16](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/03_redis%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6-16.png)

![03_redis配置文件-17](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/03_redis%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6-17.png)

![03_redis配置文件-18](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/03_redis%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6-18.png)

![03_redis配置文件-19](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/03_redis%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6-19.png)

![03_redis配置文件-20](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/03_redis%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6-20.png)

![03_redis配置文件-21](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/03_redis%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6-21.png)

