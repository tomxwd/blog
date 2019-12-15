---
title: 12_Netty_异步模型
date: 2019-12-15 09:56:01
tags: 
 - Netty
categories:
 - Netty
---

# 12_Netty_异步模型

## 基本介绍

1. 异步的概念同同步相对，当一个异步过程调用发出后，调用者不能立刻得到结果。实际处理这个调用的组件在完成后，通过状态、通知和回调来通知调用者；
2. Netty中的I/O操作是异步的，包括Bind、Write、Connect等操作会简单的返回一个ChannelFuture；
3. 调用者并不能立刻获得结果，而是通过**Future-Listener机制**，用户可以方便的**主动获取或者通过通知机制**获得IO操作结果；
4. Netty的异步模型是建立在future和callback之上的。callback就是回调。重点说Future，它的核心思想是：**假设一个方法fun，计算过程可能非常耗时，等待fun返回显然不合适。那么可以再调用fun的时候，立马返回一个Future，后续可以通过Future去监控方法fun的处理过程（即：Future-Listener机制）**；

### Future说明

1. 表示异步的执行结果，可以通过它提供的方法来检查执行是否完成。比如检索计算等；
2. ChannelFuture是一个接口，`public interface ChannelFuture extends Future<Void>`
   - 可以添加监听器，当监听的事件发生的时候，就会通知到监听器



## 工作原理![工作原理示意图](12_Netty_%E5%BC%82%E6%AD%A5%E6%A8%A1%E5%9E%8B/image-20191215101118632.png)

**说明：**

1. 在使用Netty进行编程的时候，拦截操作和转换出入栈数据只需要你提供callback或利用future即可。这使得**链式操作**简单、高效、并有利于编程可重用的、通用的代码；
2. Netty框架的目标就是让你的业务逻辑从网络基础应用编码中分离出来、解脱出来；

![链式操作示意图](12_Netty_%E5%BC%82%E6%AD%A5%E6%A8%A1%E5%9E%8B/image-20191215102434379.png)



## Future-Listener机制

1. 当Future对象刚刚创建的时候，处于非完成状态，调用者可以通过返回的ChannelFuture来获取操作执行的状态，注册监听函数来执行完成后的操作；

2. 常见有如下操作：

   - 通过isDone方法来判断当前操作是否完成；
   - 通过isSuccess方法来判断已完成的当前操作是否成功；
   - 通过getCause方法来获取已完成的当前操作失败的原因；
   - 通过isCancelled方法来判断已完成的当前操作是否被取消；
   - 通过addListener方法来注册监听器，当操作已完成（isDone方法返回完成），将会通知指定的监听器，如果Future对象已完成，则通知指定的监听器；

3. 举例说明：

   演示：绑定端口是异步操作，当绑定操作处理完，将会调用响应的监听器处理逻辑

   ```java
   // 给cf注册监听器，监控我们关心的事件
   cf.addListener(future -> {
       if(cf.isSuccess()){
           System.out.println("监听端口6688成功！");
       }else{
           System.out.println("监听端口6688失败！");
       }
   });
   ```

**小结：**

相比传统阻塞I/O，执行I/O操作后线程会被阻塞住，直到操作完成；异步处理的好处是不会造成线程阻塞，线程在I/O操作期间可以执行别的程序，在高并发情形下回更加稳定以及有更高的吞吐量；

