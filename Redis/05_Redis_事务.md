---
title: 05_Reids_事务
date: 2019-08-08 11:16:29
tags: 
 - Redis
categories:
 - NoSQL
 - Redis
---

# 05_Reids_事务

## Redis事务是什么

- 可以一次执行多个命令，本质是一组命令的集合。一个事务中的所有命令都会序列化，**按顺序地串行化执行，而不会被其他命令插入，不许加塞**

- 官网

  ![05_redis事务-1](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/05_redis%E4%BA%8B%E5%8A%A1-1.png)

## Redis事务能干嘛

一个队列中，一次性、顺序性、排他性的执行一系列命令。

## 怎么用Redis事务

![05_redis事务-2](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/05_redis%E4%BA%8B%E5%8A%A1-2.png)

![05_redis事务-3](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/05_redis%E4%BA%8B%E5%8A%A1-3.png)

### 常用命令

| 序号 | 命令及描述                                                   |
| :--- | :----------------------------------------------------------- |
| 1    | [DISCARD](https://www.runoob.com/redis/transactions-discard.html)  取消事务，放弃执行事务块内的所有命令。 |
| 2    | [EXEC](https://www.runoob.com/redis/transactions-exec.html)  执行所有事务块内的命令。 |
| 3    | [MULTI](https://www.runoob.com/redis/transactions-multi.html)  标记一个事务块的开始。 |
| 4    | [UNWATCH](https://www.runoob.com/redis/transactions-unwatch.html)  取消 WATCH 命令对所有 key 的监视。 |
| 5    | [WATCH key [key ...\]](https://www.runoob.com/redis/transactions-watch.html)  监视一个(或多个) key ，如果在事务执行之前这个(或这些) key 被其他命令所改动，那么事务将被打断。 |

### 正常执行

![05_redis事务-4](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/05_redis%E4%BA%8B%E5%8A%A1-4.png)

### 放弃事务

![05_redis事务-5](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/05_redis%E4%BA%8B%E5%8A%A1-5.png)

### 全体连坐

![05_redis事务-6](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/05_redis%E4%BA%8B%E5%8A%A1-6.png)

### 冤头债主

![05_redis事务-7](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/05_redis%E4%BA%8B%E5%8A%A1-7.png)

### watch监控

#### 悲观锁/乐观锁/CAS(Check And Set)

- 悲观锁

  ![05_redis事务-8](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/05_redis%E4%BA%8B%E5%8A%A1-8.png)

- 乐观锁

  ![05_redis事务-9](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/05_redis%E4%BA%8B%E5%8A%A1-9.png)

- CAS



1. 初始化信用卡可用余额和欠额

   ```shell
   set balance 100
   set debt 0
   ```

2. 无加塞篡改，先监控再开启multi，保证两笔金额变动在同一个事务内。

   ![05_redis事务-10](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/05_redis%E4%BA%8B%E5%8A%A1-10.png)

3. 有加塞篡改

   一处watch balance，另一处进行set balance 800操作，然后第一处又执行MULTI，开始事务，最后失败。

   监控了key，如果key被修改了，后面一个事务的执行失效。

   watch：监视一个或多个key，如果事务执行之前，这些key被改动，事务将被打断。

4. unwatch

   取消watch命令对所有key的监视。

5. **一旦执行了exec，之前加的监控锁都会被取消掉**

6. 小结

   - watch命令，类似乐观锁，事务提交的时候，如果key的值已经**被别的客户端改变**，比如某个list已经被别的客户端push/pop过了，整个事务队列都不会被执行。
   - 通过watch命令在事务执行之前监控了多个key，倘若在watch之后有任何key的值发生了变化，exec命令执行的事务都将被放弃，同时返回Nullmulti-bulk应答以通知调用者事务执行失败。

## 3阶段

- 开启：以MULTI开始一个事务。
- 入队：将多个命令入队到事务中，接到这些命令并不会立即执行，而是放到等待执行的事务队列里面。
- 执行：由EXEC命令触发事务。

## 3特性

![05_redis事务-11](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/redis/05_redis%E4%BA%8B%E5%8A%A1-11.png)

- 单独的隔离操作：事务中的所有命令都会序列化、按顺序地执行。事务在执行的过程中，不会被其他客户端发来的命令请求所打断。
- 没有隔离级别的概念：队列中的命令没有提交之前都不会实际的被执行，因为事务提交前任何指令都不会被实际执行。也就不存在“事务内的查询要看到事务里的更新，在事务处查询不能看到”，这个让人万分头痛的问题。
- 不保证原子性：redis同一个事务中如果有一条命令执行失败，其后的命令仍然会被执行，没有回滚。