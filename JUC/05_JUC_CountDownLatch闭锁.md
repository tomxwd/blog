---
title: 05_JUC_CountDownLatch闭锁
date: 2019-08-26 14:22:10
tags: 
 - Java
 - JUC
categories:
 - JUC
---

# 05_JUC_CountDownLatch闭锁

## CountDownLatch闭锁

闭锁，在完成某些运算时，只有其他线程的运算全部完成，当前运算才继续执行；

- Java5.0在java.util.concurrent包中提供了多种并发容器类来改进同步容器的性能；
- CountDownLatch一个同步辅助类，在完成一组正在其他线程中执行的操作之前，它允许一个或多个线程一直等待；
- 闭锁可以延迟线程的进度直到其到达终止状态，闭锁可以用来确保某些活动知道其他活动都完成才继续执行：
  - 确保某个计算在其他需要的所有资源都被初始化之后才继续执行；
  - 确保某个服务在其他依赖的所有其他服务都已经启动之后才启动；
  - 等待直到某个操作所有参数者都准备就绪再继续执行。



## 测试

下面的需求是当所有线程执行完，计算总时间：

代码：

```java
/**
 * TestCountDownLatch：闭锁，在完成某些运算时，只有其他线程的运算全部完成，当前运算才继续执行
 */
public class TestCountDownLatch {

    public static void main(String[] args) {
        final CountDownLatch latch = new CountDownLatch(5);
        LatchDemo ld = new LatchDemo(latch);
        long start = System.currentTimeMillis();
        for (int i = 0; i < 5; i++) {
            new Thread(ld).start();
        }
        try {
            latch.await();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        long end = System.currentTimeMillis();
        System.out.println("耗费时间为："+(end-start));
    }

}

class LatchDemo implements Runnable{

    private CountDownLatch latch;

    public LatchDemo(CountDownLatch latch){
        this.latch = latch;
    }

    public void run() {

        synchronized (this) {
            try {
                for (int i = 0; i < 50000; i++) {
                    if(i%2==0){
                        System.out.println("i = " + i);
                    }
                }
            } finally {
                latch.countDown();
            }
        }
    }
}
```



