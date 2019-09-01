---
title: 08_JUC_生产者消费者案例-虚假唤醒
date: 2019-08-26 15:27:09
tags: 
 - Java
 - JUC
categories:
 - JUC
---

# 08_JUC_生产者消费者案例-虚假唤醒

等待唤醒机制代码：

```java
/**
 * 生产者和消费者案例
 */
public class TestProductorAndConsumer {

    public static void main(String[] args) {
        Clerk clerk = new Clerk();
        Productor productor = new Productor(clerk);
        Consumer consumer = new Consumer(clerk);
        new Thread(productor,"生产者A").start();
        new Thread(consumer,"消费者B").start();
    }

}

/**
 * 店员
 */
class Clerk{

    private int product = 0;

    // 进货
    public synchronized void get(){
        if(product>=10){
            System.out.println("产品已满！");
            try {
                this.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }else{
            System.out.println(Thread.currentThread().getName()+":"+(++product));
            this.notifyAll();
        }
    }

    // 卖货
    public synchronized void sale(){
        if(product<=0){
            System.out.println("缺货！");
            try {
                this.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }else{
            System.out.println(Thread.currentThread().getName()+":"+(product--));
            this.notifyAll();
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

改进一下代码，暴露一下问题所在：

让产品只能有一个，不能再多，多了就wait，以及生产者需要0.2秒才生产一个产品，而消费者没有，因此消费者会消费的快。

```java
/**
 * 生产者和消费者案例
 */
public class TestProductorAndConsumer {

    public static void main(String[] args) {
        Clerk clerk = new Clerk();
        Productor productor = new Productor(clerk);
        Consumer consumer = new Consumer(clerk);
        new Thread(productor,"生产者A").start();
        new Thread(consumer,"消费者B").start();
    }

}

/**
 * 店员
 */
class Clerk{

    private int product = 0;

    // 进货
    public synchronized void get(){
        if(product>=1){
            System.out.println("产品已满！");
            try {
                this.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }else{
            System.out.println(Thread.currentThread().getName()+":"+(++product));
            this.notifyAll();
        }
    }

    // 卖货
    public synchronized void sale(){
        if(product<=0){
            System.out.println("缺货！");
            try {
                this.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }else{
            System.out.println(Thread.currentThread().getName()+":"+(product--));
            this.notifyAll();
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

发现程序并没有终止。

分析原因：

原因在else上，由于消费者消费的快（生产者生产一个需要0.2s）；

这时候消费者循环剩下最后一次，生产者剩下最后两次，且产品为0个，消费者抢到了资源，开始消费，发现缺货，wait；这时候生产者获得了资源，生产一个产品后唤醒，这时候又到了消费者跟生产者同时抢夺资源，如果又被消费者抢到了资源，那么就会执行完消费的方法（循环次数为0了，不会再执行消费方法），消费者就结束，这时候生产者获得资源，然而产品已经有一个了，所以进行wait等待，这时候没有人再来唤醒，因此出现了程序没终止的现象。



解决办法：

去掉else，让程序继续往下执行；

```java
/**
 * 生产者和消费者案例
 */
public class TestProductorAndConsumer {

    public static void main(String[] args) {
        Clerk clerk = new Clerk();
        Productor productor = new Productor(clerk);
        Consumer consumer = new Consumer(clerk);
        new Thread(productor,"生产者A").start();
        new Thread(consumer,"消费者B").start();
    }

}

/**
 * 店员
 */
class Clerk{

    private int product = 0;

    // 进货
    public synchronized void get(){
        if(product>=1){
            System.out.println("产品已满！");
            try {
                this.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        System.out.println(Thread.currentThread().getName()+":"+(++product));
        this.notifyAll();
    }

    // 卖货
    public synchronized void sale(){
        if(product<=0){
            System.out.println("缺货！");
            try {
                this.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        System.out.println(Thread.currentThread().getName()+":"+(product--));
        this.notifyAll();
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



虽然去掉了else，但是其实还是存在问题：多个消费者多个生产者的情况呢？

```java
new Thread(productor,"生产者A").start();
new Thread(consumer,"消费者B").start();
new Thread(productor,"生产者C").start();
new Thread(consumer,"消费者D").start();
```

同时唤醒会导致两个消费者在wait的时候，同时消费了资源，因此结果出现负数。这种问题称之为**虚假唤醒**。

jdk中有给出wait的说明，其中就有解决方案，只要把if改为while即可（**为了避免虚假唤醒问题，应该总是把wait放在循环中**）：

```java
// 进货
public synchronized void get(){
    while (product>=1){
        System.out.println("产品已满！");
        try {
            this.wait();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
    System.out.println(Thread.currentThread().getName()+":"+(++product));
    this.notifyAll();
}

// 卖货
public synchronized void sale(){
    while(product<=0){
        System.out.println("缺货！");
        try {
            this.wait();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
    System.out.println(Thread.currentThread().getName()+":"+(product--));
    this.notifyAll();
}
```

