---
title: 02_Netty_BIO编程
date: 2019-11-27 21:53:25
tags: 
 - Netty
categories:
 - Netty
---

# 02_Netty_BIO编程

## I/O模型

1. I/O模型简单的理解：就是用怎么样的通道进行数据的发送和接收，很大程度上决定了程序通信的性能；

2. Java共支持3种网络编程模型/IO模式：BIO、NIO、AIO

   - BIO：**同步并阻塞**（传统阻塞型），服务器实现模式为一个连接一个线程，即客户端有连接请求时服务器端就需要启动一个线程进行处理，如果这个连接不做任何事情会造成不必要的线程开销；

     ![BIO模式](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/Netty/02BIO%E6%A8%A1%E5%BC%8F.png)

   - NIO：**同步非阻塞**，服务器实现模式为一个线程处理多个请求（连接），即客户端发送的连接请求都会注册到多路复用器上，多路复用器轮询到连接有I/O请求就进行处理；![NIO模式](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/Netty/02NIO%E6%A8%A1%E5%BC%8F.png)

   - AIO(NIO.2)：**异步非阻塞**，AIO引入异步通道的概念，采用了Proactor模式，简化了程序的编写，有效的请求才启动线程，它的特点是先由操作系统完成后才通知服务端程序启动线程去处理，一般适用于连接数较多且连接时间较长的应用；



## BIO、NIO、AIO适用场景分析

1. BIO方式适用于连接数目比较小且固定的架构，这种方式对服务器资源要求比较高，并发局限于应用中，JDK1.4以前的唯一选择，但程序简单易理解；
2. NIO方式适用于**连接数目多且连接比较短**（轻操作）的架构，比如聊天服务器，弹幕系统，服务器间通讯等。编程较复杂，JDK1.4开始支持；
3. AIO方式使用与**连接数目多且连接比较长**（重操作）的架构，比如相册服务器，充分调用OS参与并发操作，编程比较复杂，JDK7开始支持。



## Java BIO 基本介绍

1. Java BIO就是传统的Java io编程，其相关的类和接口在java.io
2. BIO(blocking I/O)：**同步阻塞**，服务器实现模式为一个连接一个线程，即客户端有连接请求时服务器端就需要启动一个线程进行处理，如果这个连接不做任何事情会造成不必要的线程开销，可以通过线程池机制改善。
3. BIO方式适用于连接数目比较小且固定的架构，这种方式对服务器资源要求比较高，并发局限于应用中，JDK1.4以前的唯一选择，程序简单易理解。



### BIO编程简单流程

1. 服务器端启动一个ServerSocket；
2. 客户端启动Socket对服务器进行通信，默认情况下服务器端需要为每个客户端创建一个线程与之通信；
3. 客户端发出请求后，先咨询服务器是否有线程响应，如果没有则等待、或超时、或被拒绝；
4. 如果有响应，客户端线程会等待请求结束后，再继续执行；



### Java BIO 应用实例

实例说明：

1. 使用BIO模型编写一个服务器端，监听6666端口，当有客户端连接的时候，就自动创建一个线程与之通信；
2. 要求使用线程池机制改善，可以连接多个客户端；
3. 服务器端可以接收客户端发送的数据，需要用telnet的方式；
   - 用telent ip port连接服务器
   - 连接后用Ctrl+]进入Client
   - send 内容，来发送数据到服务器

思路：

1. 创建一个线程池；
2. 如果有客户端连接，就创建一个线程与之通信（单独写一个方法）；

```java
public class BIOServer {
    public static void main(String[] args) throws IOException {
        // 线程池机制
        ExecutorService newCachedThreadPool = Executors.newCachedThreadPool();

        // 创建ServerSocket
        ServerSocket serverSocket = new ServerSocket(6666);
        System.out.println("服务器启动了");
        while (true) {
            // 监听，等待客户端连接
            final Socket socket = serverSocket.accept();
            System.out.println("连接到客户端了");

            // 创建一个线程与之通信
            newCachedThreadPool.execute(() -> handler(socket));
        }

    }

    // 编写一个handler方法，和客户端通讯
    public static void handler(Socket socket){
        try {
            System.out.println("线程信息 id = "+Thread.currentThread().getId()+"名字 = "+Thread.currentThread().getName());
            byte[] bytes = new byte[1024];
            // 通过socket获取输入流
            InputStream inputStream = socket.getInputStream();
            // 循环读取客户端发送的数据
            while (true){
                System.out.println("线程信息 id = "+Thread.currentThread().getId()+"名字 = "+Thread.currentThread().getName());
                int read = inputStream.read(bytes);
                if(read!=-1){
                    // 输出客户端发送的数据
                    System.out.println(new String(bytes,0,read));
                } else {
                    break;
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            System.out.println("关闭和client的连接");
            try {
                socket.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

问题分析：

1. 每个请求都需要创建独立的线程，与对应的客户端进行数据Read，业务处理，数据Write；
2. 当并发数较大的时候，需要**创建大量线程来处理连接**，系统资源占用较大；
3. 连接建立后，如果当前线程暂时没有数据可读，则线程就阻塞在Read操作上，造成线程资源浪费；