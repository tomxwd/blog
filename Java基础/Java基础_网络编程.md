---
title: Java基础_网络编程
date: 2019-12-15 20:37:59
tags: 
 - Java
categories:
 - Java
---

# Java基础_网络编程

> [OSI 七层模型和TCP/IP模型及对应协议（详解）](https://blog.csdn.net/qq_41923622/article/details/85805003)

![在这里插入图片描述](Java%E5%9F%BA%E7%A1%80_%E7%BD%91%E7%BB%9C%E7%BC%96%E7%A8%8B/20190105164025264.png)



## 端口

端口表示计算机上一个程序的进程；

- 不同的进程有不同的端口号，用来区分软件
- 被规定0-65535
- 分为TPC以及UDP端口，也就是能有65536*2的进程；



### 端口分类

- 公有端口0~1023

  - Http：80
  - Https：443
  - FTP：21
  - SSH：22
  - telnet：23

- 程序注册端口：1024~49151，分配用户或者程序

  - Tomcat：8080
  - MySQL：3306
  - Oracle：1521

- 动态、私有端口：49152~65535

- 常见的命令：

  ```shell
  netstat -ano	# 查看所有端口
  netstat -ano|findstr "5900"		# 查看指定的端口
  tasklist|findstr "8696"		# 查看指定端口的进程	
  ```



## 通信协议

协议：约定

**网络通信协议：**速率，传输码率，传输结构，控制



### TCP/IP协议簇

TCP/IP协议簇是一组协议，包括很多

重要的两个：

- TCP：用户传输协议
- UDP：用户数据报协议

出名的两个：

- TCP：用户传输协议
- IP：网络互连协议



### TCP UDP对比

TCP：打电话

- 连接，稳定

- **三次握手和四次挥手**

  最少需要三次，保证稳定连接！

  ```sequence
  title: 三次握手
  participant A
  participant B
  A->B: 你瞅啥?
  B->A: 瞅你咋地?
  A->B: 干一场!
  ```

  ```sequence
  title: 四次挥手
  participant A
  participant B
  A->B: 我要断开了
  B->A: 我知道你要断开了
  B->A: 你真的断开了吗
  A->B: 我断开了
  ```

  

- 客户端、服务端

- 传输完成、释放连接，效率低

UDP：发短信

- 不连接，不稳定
- 客户端、服务端：没有明确的界限

- 不管有没有准备好，都可以发给你
- DDOS：洪水攻击！（饱和攻击）



## TCP

客户端：

1. 连接服务器Socket
2. 发送消息

```java
public class TcpClientDemo01 {

    public static void main(String[] args) {
        Socket socket = null;
        OutputStream os = null;
        try {
            // 1. 客户端要知道服务器的地址
            InetAddress serverIp = InetAddress.getByName("127.0.0.1");
            // 2. 端口号
            int port = 9999;
            // 3. 创建一个Socket连接
            socket = new Socket(serverIp, port);
            // 4. 发送消息，IO流
            os = socket.getOutputStream();
            os.write("你好，Hello".getBytes());
        } catch (UnknownHostException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if(os!=null){
                try {
                    os.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if (socket != null) {
                try {
                    socket.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }

}
```

服务端：

1. 建立服务的端口ServerSocket
2. 等待用户的连接，通过accept
3. 接收用户的消息

```java
public class TcpServerDemo01 {

    public static void main(String[] args) {
        ServerSocket serverSocket = null;
        Socket accept = null;
        InputStream is = null;
        ByteArrayOutputStream baos = null;
        try {
            // 1. 我得有一个地址
            serverSocket = new ServerSocket(9999);
            // 2. 等待客户端连接
            accept = serverSocket.accept();
            // 3. 读取客户端消息
            is = accept.getInputStream();
//            byte[] buffer = new byte[1024];
//            int len;
//            while ((len=is.read(buffer))!=-1){
//                String s = new String(buffer, 0, len);
//                System.out.println("msg = " + s);
//            }
            // 管道流
            baos = new ByteArrayOutputStream();
            byte[] buffer = new byte[1024];
            int len;
            while ((len = is.read(buffer)) != -1) {
                baos.write(buffer, 0, len);
            }
            System.out.println(baos.toString());
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            // 关闭资源
            if (baos != null) {
                try {
                    baos.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if (is != null) {
                try {
                    is.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if (accept != null) {
                try {
                    accept.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if (serverSocket != null) {
                try {
                    serverSocket.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }

}
```



## TCP文件上传

服务器端：

```java
public class TcpServerDemo02 {
    public static void main(String[] args) throws IOException {
        // 创建服务
        ServerSocket serverSocket = new ServerSocket(9999);
        // 监听
        Socket accept = serverSocket.accept();
        // 获取输入流
        InputStream is = accept.getInputStream();
        // 文件输出
        FileOutputStream fos = new FileOutputStream(new File("pic2.png"));
        byte[] buffer = new byte[1024];
        int len;
        while ((len = is.read(buffer)) != -1) {
            fos.write(buffer, 0, len);
        }
        // 通知客户端文件接收完毕
        OutputStream os = accept.getOutputStream();
        os.write("我接收完毕了，可以断开".getBytes(Charset.forName("utf-8")));
        // 关闭资源
        fos.close();
        is.close();
        accept.close();
        serverSocket.close();

    }
}
```

客户端：

```java
public class TcpClientDemo02 {

    public static void main(String[] args) throws IOException {
        // 创建一个socket连接
        Socket socket = new Socket("localhost", 9999);
        // 创建输出流
        OutputStream os = socket.getOutputStream();
        // 文件流
        FileInputStream fis = new FileInputStream(new File("pic.png"));
        // 写出文件
        byte[] buffer = new byte[1024];
        int len;
        while ((len = fis.read(buffer)) != -1) {
            os.write(buffer, 0, len);
        }
        // 通知服务器已经输出完毕
        socket.shutdownOutput();
        // 确定服务器接收完毕才可以连接
        InputStream is = socket.getInputStream();
        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        byte[] buffer2 = new byte[1024];
        int len2;
        while ((len2 = is.read(buffer2)) != -1) {
            baos.write(buffer2, 0, len2);
        }
        System.out.println(baos.toString());
        // 关闭资源
        baos.close();
        is.close();
        fis.close();
        os.close();
        socket.close();
    }

}
```







