---
title: 08_Netty_Netty高性能架构设计
date: 2019-12-02 21:17:57
tags: 
 - Netty
categories:
 - Netty
---

# 08_Netty_Netty高性能架构设计

## 线程模型基本介绍

1. 不同的线程模式，对程序的性能有很大影响，为了搞清Netty线程模式，系统看看各个线程模式，最后看看Netty线程模型有何优越性；
2. 目前存在的线程模型有：
   - 传统阻塞I/O服务模型
   - Reactor模式
3. 根据Reactor的数量和处理资源线程池的数量不同，有3种典型的实现
   - 单Reactor单线程
   - 单Reactor多线程
   - 主从Reactor多线程
4. Netty线程模式**（Netty主要基于主从Reactor多线程模型做了一些改进，其中主从Reactor多线程模型有多个Reactor）**；



### 传统阻塞I/O服务模型

![传统阻塞IO服务模型工作原理图](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/Netty/08%E4%BC%A0%E7%BB%9F%E9%98%BB%E5%A1%9EIO%E6%9C%8D%E5%8A%A1%E6%A8%A1%E5%9E%8B%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86%E5%9B%BE.png)

黄色的框表示对象，蓝色的框代表线程，白色的框标识方法（API）；

模型特点：

1. 采用阻塞I/O模型获取输入的数据
2. 每个连接都需要独立的线程完成数据的输入、业务处理、数据返回

问题分析：

1. 并发数量大的时候，需要创建大量的线程，占用很大的系统资源；
2. 连接创建后，如果当前线程暂时没有数据可读，该线程会阻塞到read操作上，造成线程资源浪费；



### Reactor模式

Reactor对应的叫法：

1. **反应器模式**
2. **分发者模式**（Dispatcher）
3. **通知者模式**（Notifier）

针对传统阻塞I/O服务模型的2个缺点，解决方案：

1. **基于I/O复用模型**：多个连接公用一个阻塞对象，应用程序只需要在一个阻塞对象等待，无需阻塞等待所有连接。当某个连接有新的数据可以处理的时候，操作系统通知应用程序，线程从阻塞状态返回，开始进行业务处理；
2. **基于线程池复用线程资源：**不必再为每个连接创建线程，将连接完成后的业务处理任务分配给线程进行处理，一个线程可以处理多个连接的业务；

![Reactor模式](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/Netty/08Reactor%E6%A8%A1%E5%BC%8F.png)

整体设计理念（可能有不同的实现，有些许区别，但大体一致）：

1. Reactor模式，通过一个或多个输入同时传递给服务器的模式（基于事件驱动）
2. 服务器端的程序处理传入的多个请求并将它们同步分派到相应的处理线程中去，因此Reactor模式也叫Dispatch模式（分发者模式）
3. Reactor模式使用了IO复用监听事件，收到事件后，分发给某个线程（进程），这点就是网络服务高并发处理的关键；



#### Reactor模式核心组成

1. **Reactor**：Reactor在一个单独的线程中运行，负责监听和分发事件，分发给适当的处理程序来对IO事件做出反应。它就像公司的电话接线员，接听来自客户的电话并将线路转移到适当的联系人；
2. **Handlers**：处理程序执行I/O事件要完成的实际事件，类似于客户想要与之交谈的公司中的实际官员。Reactor通过调度适当的处理程序来响应I/O事件，处理程序执行非阻塞操作；



#### Reactor模式分类

根据Reactor的数量和处理资源池线程的数量不同，有3种典型的实现：

1. 单Reactor单线程
2. 单Reactor多线程
3. 主从Reactor多线程



##### 单Reactor单线程

![单Reactor单线程](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/Netty/08%E5%8D%95Reactor%E5%8D%95%E7%BA%BF%E7%A8%8B.png)

方案说明：

1. Select是前面I/O复用模型介绍的标准网络编程API，可以实现应用程序通过一个阻塞对象监听多路连接请求；
2. Reactor对象通过Select监控客户端请求事件，收到事件后通过Dispatch进行分发；
3. 如果是建立连接请求事件，则由Acceptor通过Accept处理连接请求，然后创建一个Handler对象处理连接完成后的后续业务处理；
4. 如果不是建立连接时间，则Reactor会分发调用连接对应的Handler来响应；
5. Handler会完成Read->业务处理->Send的完整业务流程；

综合实例：

服务器端用一个线程通过多路复用搞定所有的IO操作（包括连接、读、写等），编码简单，清晰明了，但是如果客户端连接数量过多，将无法支撑，前面的NIO案例就属于这种模型；

方案优缺点分析：

1. **优点：**模型简单，没有多线程、进程通信、竞争的问题，全部在一个线程中完成；
2. **缺点：**
   - 性能问题，只有一个线程，无法完全发挥多核CPU的性能。Handler在处理某个连接上的业务时，整个进程无法处理其他连接事件，很容易导致性能瓶颈；
   - 可靠性问题，线程意外终止，或者进入死循环，会导致整个系统通信模块不可用，不能接收和处理外部消息，造成节点故障；
3. **使用场景：**客户端的数量有限，业务处理非常快速，比如Redis在业务处理的事件复杂度O(1)的情况；



##### 单Reactor多线程

![单Reactor多线程](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/Netty/08%E5%8D%95Reactor%E5%A4%9A%E7%BA%BF%E7%A8%8B.png)

方案说明：

1. Reactor对象通过select监控客户端请求事件，收到事件后通过Dispatch进行分发；；
2. 如果是建立连接的请求，则由Acceptor通过accept处理连接请求，然后创建一个Handler对象处理完成连接后的各种事件；
3. 如果不是连接请求，则由Reactor对象进行分发调用连接对应的Handler来处理；
4. Handler只负责响应事件，不做具体的业务处理，通过read读取数据后，会分发给后面的Worker线程池的某个线程处理业务；
5. Worker线程池会分配独立线程完成真正的业务，并将结果返回给Handler；
6. Handler收到响应后，通过send将结果返回给Client；

方案优缺点分析：

1. **优点：**可以充分的利用多核CPU的处理能力；
2. **缺点：**多线程会数据共享和访问比较复杂，Reactor处理所有的事件的监听和响应，在单线程运行中，在高并发场景中容易出现性能瓶颈；



##### 主从Reactor多线程

![主从Reactor多线程](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/Netty/08%E4%B8%BB%E4%BB%8EReactor%E5%A4%9A%E7%BA%BF%E7%A8%8B.png)

方案说明：

1. Reactor主线程MainReactor对象通过select监听连接事件，收到事件后，通过Acceptor处理连接事件；
2. 当Acceptor处理连接事件后，MainReactor将连接分配给SubReactor；
3. SubReactor将连接加入到连接队列进行监听，并创建Handler进行各种事件处理；
4. 当有新的事件发生时，SubReactor就会调用对应的Handler进行处理；
5. Handler通过read读取数据，分发给后面的Worker线程处理；
6. Worker线程池会分配独立的Worker线程进行业务处理，并返回结果到Handler；
7. Handler收到响应的结果后，再通过send方法返回给Client；
8. Reactor主线程可以对应多个Reactor子线程，即MainReactor可以关联多个SubReactor；



Scalable IO in Java对Multiple Reactors的原理图解：

![ScalableIOinJava](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/Netty/08ScalableIOinJava.png)

方案优缺点说明：

1. **优点：**
   - 父线程和子线程的数据交互简单、职责明确，父线程只需要接收新连接，子线程完成后续的业务处理；
   - 父线程与子线程的数据交互简单，Reactor主线程只需要把新连接传给子线程，子线程无需返回数据；
2. **缺点：**
   - 编程复杂度较高；

结合实例：这种模型在许多项目中广泛使用，包括Nginx主从Reactor多线程模型，Memcached主从多线程，Netty主从多线程模型的支持；



#### Reactor模式小结

三种模式用生活案例来理解：

1. 单Reactor单线程：前台接待员和服务员是同一个人，全程为顾客服务；
2. 单Reactor多线程：一个前台接待员，多个服务员，接待员只需要负责接待，服务员负责服务；
3. 主从Reactor多线程，多个前台接待员，多个服务生；



Reactor模式具有如下的优点：

1. 响应快，不必为单个同步时间所阻塞，虽然Reactor本身依然是同步的；
2. 可以最大程度的避免复杂的多线程以及同步问题，并且避免了多线程/进程的切换开销；
3. 扩展性好，可以方便的通过增加Reactor实例个数来充分利用CPU资源；
4. 复用性好，Reactor模型本身与具体事件处理逻辑无关，具有很高的复用性；













