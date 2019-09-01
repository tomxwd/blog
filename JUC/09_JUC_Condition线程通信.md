---
title: 09_JUC_Condition线程通信
date: 2019-08-26 16:34:16
tags: 
 - Java
 - JUC
categories:
 - JUC
---

# 09_JUC_Condition线程通信

改造之前用synchronized的方法，改用同步锁Lock来实现，而Lock有自己的wait方式，就是Condition；

## Condition

- Condition接口描述了可能会与锁有关联的条件变量。这些变量在用法上与使用Object.wait访问的隐式监视器类似，但提供了更加强大的功能。需要特别指出的是，单个Lock可能与多个Condition对象关联。为了避免兼容性问题，Condition方法的名称与对应的Object版本中的不同。
- 在Condition对象中，与wait、notify和notifyAll方法对应的分别是await、signal和signalAll。
- Condition实例实质上被绑定到一个锁上，要为特定Lock实例获得Condition实例，请使用其newCondition()方法。

```java
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

/**
 * 生产者和消费者案例
 */
public class TestProductorAndConsumerForLock {

    public static void main(String[] args) {
        Clerk clerk = new Clerk();
        Productor productor = new Productor(clerk);
        Consumer consumer = new Consumer(clerk);
        new Thread(productor,"生产者A").start();
        new Thread(consumer,"消费者B").start();
        new Thread(productor,"生产者C").start();
        new Thread(consumer,"消费者D").start();
    }

}

/**
 * 店员
 */
class Clerk{

    private int product = 0;

    private Lock lock = new ReentrantLock();

    private Condition condition = lock.newCondition();

    // 进货
    public void get(){
        lock.lock();
        try {
            while (product>=1){
                System.out.println("产品已满！");
                try {
                    condition.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            System.out.println(Thread.currentThread().getName()+":"+(++product));
            condition.signalAll();
        } finally {
            lock.unlock();
        }
    }

    // 卖货
    public void sale(){
        lock.lock();
        try {
            while(product<=0){
                System.out.println("缺货！");
                try {
                    condition.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            System.out.println(Thread.currentThread().getName()+":"+(product--));
            condition.signalAll();
        } finally {
            lock.unlock();
        }
    }

}

/**
 * 生产者
 */
class Productor implements Runnable{
    private Clerk clerk;

    public Productor(Clerk clerk){
        this.clerk = clerk;
    }

    public void run() {
        for (int i = 0; i < 20; i++) {
            try {
                Thread.sleep(200);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            clerk.get();
        }
    }
}

/**
 * 消费者
 */
class Consumer implements Runnable{

    private Clerk clerk;

    public Consumer(Clerk clerk) {
        this.clerk = clerk;
    }

    public void run() {
        for (int i = 0; i < 20; i++) {
            clerk.sale();
        }
    }
}
```



