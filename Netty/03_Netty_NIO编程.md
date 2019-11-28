---
title: 03_Netty_NIO编程
date: 2019-11-28 20:56:23
tags: 
 - Netty
categories:
 - Netty
---

# 03_Netty_NIO编程

## Java NIO 基本介绍

![NIO模式](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/Netty/02NIO%E6%A8%A1%E5%BC%8F.png)

![NIO模式2](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/Netty/03NIO%E6%A8%A1%E5%BC%8F2.png)

1. Java NIO 全称java non-blocking IO，是指JDK提供的新API。从JDK1.4开始，Java提供了一系列改进的输入/输出的新特性，被统称为NIO（即New IO），是**同步非阻塞**的；
2. NIO相关类都被放在java.nio包及其子包下，并且对原java.io包中的很多类进行改写。
3. NIO有三大核心部分：
   - Channel：通道
   - Buffer：缓冲区
   - Selector：选择器
4. NIO是面向**缓冲区，或者面向块**编程的。数据读取到一个它稍后处理的缓冲区，需要时可以在缓冲区中前后移动，这就增加了处理过程中的灵活性，使用它可以提供非阻塞式的高伸缩性网络；
5. Java NIO的非阻塞模式，使一个线程从某个通道发送请求或者读取数据，但是它仅能得到目前可用的数据，如果目前没有数据可用的时候，就什么都不会获取，而不是保持线程阻塞，所以直到数据变得可以读取之前，该线程可以继续做其他的事情。非阻塞写也是如此，一个线程请求写入一些数据到某个通道，但不需要等待它完全写入，这个线程同时可以去做别的事情；
6. 通俗理解：NIO是可以做到用一个线程来处理多个操作的。假设有10000个请求过来，根据实际情况，可以分配50或者100个线程来处理，不像之前的阻塞IO那样，非得分配10000个线程来做处理；
7. HTTP2.0使用了多路复用的技术，做到同一个连接并发处理多个请求，而且并发请求的数量比HTTP1.1大了好几个数量级；



## NIO Buffer的基本使用

```java
public class BasicBuffer {

    public static void main(String[] args) {
        // 举例说明Buffer的使用
        // 创建一个Buffer,大小为5，即可以存放5个int
        IntBuffer intBuffer = IntBuffer.allocate(5);

        // 向Buffer中存放数据
//        intBuffer.put(10);
//        intBuffer.put(11);
//        intBuffer.put(12);
//        intBuffer.put(13);
//        intBuffer.put(14);
        for (int i = 0; i < intBuffer.capacity(); i++) {
            intBuffer.put(i * 2);
        }
        // 如何从Buffer读取数据
        // 将Buffer转换，读写切换
        intBuffer.flip();
        while (intBuffer.hasRemaining()) {
            System.out.println(intBuffer.get());
        }
    }

}
```

主要注意：flip()进行翻转，读写模式切换；



## NIO 和 BIO的比较

1. BIO以流的方式处理数据，而NIO以块的方式处理数据。块I/O的效率比流I/O高很多；
2. BIO是阻塞的，NIO是非阻塞的；
3. BIO基于字节流和字符流进行操作，而NIO基于Channel（通道）和Buffer（缓冲区）进行操作，数据总是从通道读取到缓冲区中，或者从缓冲区写入到通道中。Selector（选择器）用于监听多个通道的事件（比如：连接请求，数据到达等），因此使用**单个线程就可以监听多个客户端通道**;



## NIO 三大核心组件关系

NIO的Selector、Channel、Buffer的关系：

![NIO三大核心组件关系图](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/Netty/03NIO%E4%B8%89%E5%A4%A7%E6%A0%B8%E5%BF%83%E7%BB%84%E4%BB%B6%E5%85%B3%E7%B3%BB%E5%9B%BE.png)

1. 每个Channel都会对应一个Buffer；
2. 一个Selector对应一个线程，一个线程对应多个Channel（连接）；
3. 该图反应了有三个Channel注册到该Selector；
4. 程序切换到哪个Channel是由事件决定的，Event是一个很重要的概念；
5. Selector会根据不同的事件，在各个通道上切换；
6. Buffer 就是一个内存块，底层是一个数组
7. 数据的读取/写入都是通过Buffer，这个和BIO不同，BIO中要么是输入流或者是输出流，不能双向流动，但是NIO的**Buffer可读可写**，需要用flip()方法切换；
8. **Channel是双向的**，可以反映底层操作系统的情况，比如Linux底层的操作系统通道就是双向的；



## 缓冲区（Buffer）

基本介绍：

- 缓冲区（Buffer）：缓冲区本质上是一个**可以读写数据的内存块**，可以理解是一个**容器对象（含数组）**，该对象提供了一组方法，可以更加轻松地使用内存块；

- 缓冲区对象内置了一些机制，能够跟踪和记录缓冲区的状态变化情况；

- Channel提供从文件、网络读取数据的渠道，但是读取或写入的数据都必须经由Buffer：

  ![缓冲区图](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/Netty/03%E7%BC%93%E5%86%B2%E5%8C%BA%E5%9B%BE.png)



### Buffer类及其子类

在NIO中，Buffer是一个顶级父类，是一个抽象类，类的层级关系图：

![Buffer类的层级关系图](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/Netty/03Buffer%E7%B1%BB%E5%B1%82%E7%BA%A7%E5%85%B3%E7%B3%BB%E5%9B%BE.png)

常用Buffer子类一览：

1. **ByteBuffer**：存储字节数据到缓冲区
2. ShortBuffer：存储字符串数据到缓冲区
3. CharBuffer：存储字符数据到缓冲区
4. IntBuffer：存储整数数据到缓冲区
5. LongBuffer：存储长整型数据到缓冲区
6. DoubleBuffer：存储小数到缓冲区
7. FloatBuffer：存储小数到缓冲区



### Buffer类数据元素信息

Buffer类定义了所有的缓冲区都具有四个属性来提供关于其所包含的数据元素信息：

- private int mark =  -1;
- private int position = 0;
- private int limit;
- private int capacity;

| 属性     | 描述                                                         |
| -------- | ------------------------------------------------------------ |
| Capacity | 容量，即可以容纳的最大数据量，在缓冲区创建时被设定并且**不能改变**； |
| Limit    | 表示缓冲区的当前终点，不能对缓冲区超过极限的位置进行读写操作。且**极限是可以被修改**的； |
| Position | 位置，下一个要被读或者写的元素的索引，**每次读写缓冲区的数据都会改变值**，为下次读写做准备； |
| Mark     | 标记                                                         |

重点关注flip()方法：

```java
public final Buffer flip() {
    limit = position;
    position = 0;
    mark = -1;
    return this;
}
```

比如一直写入数据，要读的时候，调用flip，那么就是写入多少个，读取也最多只能是多少个，因此limit=position；



### Buffer类相关方法一览

```java
public abstract class Buffer {

    // JDK1.4引入
    // 返回缓冲区的容量
    public final int capacity() {
        return capacity;
    }

    // 返回此缓冲区的位置
    public final int position() {
        return position;
    }

    // 设置此缓冲区的位置
    public final Buffer position(int newPosition) {
        if ((newPosition > limit) || (newPosition < 0))
            throw new IllegalArgumentException();
        position = newPosition;
        if (mark > position) mark = -1;
        return this;
    }

    // 返回此缓冲区的限制
    public final int limit() {
        return limit;
    }

    // 设置此缓冲区的限制
    public final Buffer limit(int newLimit) {
        if ((newLimit > capacity) || (newLimit < 0))
            throw new IllegalArgumentException();
        limit = newLimit;
        if (position > limit) position = limit;
        if (mark > limit) mark = -1;
        return this;
    }

   	// 在此缓冲区的位置设置标记
    public final Buffer mark() {
        mark = position;
        return this;
    }

    // 将此缓冲区的位置重置为以前标记的位置
    public final Buffer reset() {
        int m = mark;
        if (m < 0)
            throw new InvalidMarkException();
        position = m;
        return this;
    }

    // 清除此缓冲区，即将各个标记恢复到初始状态，但是数据并没有真正擦除
    public final Buffer clear() {
        position = 0;
        limit = capacity;
        mark = -1;
        return this;
    }

    // 反转此缓冲区
    public final Buffer flip() {
        limit = position;
        position = 0;
        mark = -1;
        return this;
    }

    // 重绕此缓冲区
    public final Buffer rewind() {
        position = 0;
        mark = -1;
        return this;
    }

    // 返回当前位置与限制之间的元素数
    public final int remaining() {
        return limit - position;
    }

    // 告知在当前位置和限制之间是否有元素
    public final boolean hasRemaining() {
        return position < limit;
    }

    // 告知此缓冲区是否为只读缓冲区
    public abstract boolean isReadOnly();

    // JDK1.6引入
    // 告知此缓冲区是否具有可访问的底层实现数组
    public abstract boolean hasArray();

    // 返回此缓冲区的底层实现数组
    public abstract Object array();

    // 返回此缓冲区底层实现数组中的第一个缓冲区元素的偏移量
    public abstract int arrayOffset();

    // 告知此缓冲区是否为直接缓冲区
    public abstract boolean isDirect();

}
```



### ByteBuffer

从前面可以看出对于Java中的基本数据类型（除布尔类型），都有一个Buffer类型与之对应，最常用的自然是ByteBuffer类（二进制数据），该类的主要方法：

```java
public abstract class ByteBuffer
    extends Buffer
    implements Comparable<ByteBuffer>
{

    // 缓冲区创建相关API
    
    // 创建直接缓冲区
    public static ByteBuffer allocateDirect(int capacity) {
        return new DirectByteBuffer(capacity);
    }

    // 设置缓冲区的初始容量
    public static ByteBuffer allocate(int capacity) {
        if (capacity < 0)
            throw new IllegalArgumentException();
        return new HeapByteBuffer(capacity, capacity);
    }

    // 构造初始化位置offset和上界length的缓冲区
    public static ByteBuffer wrap(byte[] array,
                                    int offset, int length)
    {
        try {
            return new HeapByteBuffer(array, offset, length);
        } catch (IllegalArgumentException x) {
            throw new IndexOutOfBoundsException();
        }
    }

    // 把一个数组放到缓冲区中使用
    public static ByteBuffer wrap(byte[] array) {
        return wrap(array, 0, array.length);
    }
    
    // 缓冲区存取相关API

    // 从当前位置position上get，get之后，position会自动-1
    public abstract byte get();

    // 从当前位置上put，put之后，position会自动+1
    public abstract ByteBuffer put(byte b);
	
    // 从绝对位置get
    public abstract byte get(int index);
	
    // 从绝对位置上put
    public abstract ByteBuffer put(int index, byte b);

}
```







