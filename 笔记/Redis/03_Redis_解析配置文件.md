---
2019-08-07 15:06:48

---



# Redis_解析配置文件

## 1. 配置文件的位置

### 地址

在redis解压目录下就有一个redis.conf文件，最好拷贝出来使用。

## 为什么要拷贝出来使用

并不能保证一次就能修改正确，且可能需要配置不一样的情况。

## 2. Units单位

![1565166260976](../数据结构/数据结构图解/1565166260976.png)

## 3. INCLUDES包含

![1565166446163](../数据结构/数据结构图解/1565166446163.png)

## 4. GENERAL通用

![1565166477747](../数据结构/数据结构图解/1565166477747.png)

- Daemonize

- Pidfile

- Port

- Tcp-backlog

  ![1565166589868](../数据结构/数据结构图解/1565166589868.png)

- Timeout

- Bind

- Tcp-keepalive

  ![1565166702715](../数据结构/数据结构图解/1565166702715.png)

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

     ![1565226312796](../数据结构/数据结构图解/1565226312796.png)

   - 禁用

     ![1565226343187](../数据结构/数据结构图解/1565226343187.png)

2. stop-writes-on-bgsave-error

   ![1565226587777](../数据结构/数据结构图解/1565226587777.png)

3. rdbcompression

   ![1565226696602](../数据结构/数据结构图解/1565226696602.png)

4. rdbchecksum

   ![1565226719580](../数据结构/数据结构图解/1565226719580.png)

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

  ![1565169217536](../数据结构/数据结构图解/1565169217536.png)

  - noeviction：永不过期

- maxmemory-samples 5

  ![1565169282433](../数据结构/数据结构图解/1565169282433.png)

- replica-ignore-maxmemory yes

## 9. APPEND ONLY MODE追加

- appendonly yes/no ：是否开启aof配置

- appendfilename 文件名

- appendfsync

  同步策略：

  - always：同步持久化，每次发生数据变更都会被立即记录到磁盘，性能较差，但是数据完整性比较好；
  - everysec：**出厂默认推荐**，异步操作，每秒记录，如果一秒内宕机，会有数据丢失；
  - No：从不；

  ![1565231205560](../数据结构/数据结构图解/1565231205560.png)

- no-appendfsync-on-rewrite：重写时是否可以运用appendfsync，用默认no即可，保证数据安全性。

- auto-aof-rewrite-min-size：设置重写的基准值，

- auto-aof-rewrite-percentage：设置重写的基准值，

## 10. 常见配置redis.conf介绍

![1565169374720](../数据结构/数据结构图解/1565169374720.png)

![1565169440266](../数据结构/数据结构图解/1565169440266.png)

![1565169479980](../数据结构/数据结构图解/1565169479980.png)

![1565169489151](../数据结构/数据结构图解/1565169489151.png)

![1565169511130](../数据结构/数据结构图解/1565169511130.png)

![1565169572267](../数据结构/数据结构图解/1565169572267.png)

![1565169582122](../数据结构/数据结构图解/1565169582122.png)

![1565169590253](../数据结构/数据结构图解/1565169590253.png)