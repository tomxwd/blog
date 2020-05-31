---
2019-08-26 17:00:42

---

#

ReadWriteLock：读写锁
写写/读写：需要**互斥**
读读：不需要**互斥**

ReadWriteLock维护了两个锁，一个是读的锁，一个是写的锁，读的锁可以被多个读线程并发的持有，而写的锁是独占（排他）的。

方法：

- Lock readLock()
- Lock writeLock()

```java
package top.tomxwd.juc;

import java.util.concurrent.locks.ReadWriteLock;
import java.util.concurrent.locks.ReentrantReadWriteLock;

/**
 * ReadWriteLock：读写锁
 * 写写/读写：需要互斥
 * 读读：不需要互斥
 */
public class TestReadWriteLock {

    /**
     * 1个线程去写，100个线程去读
     * @param args
     */
    public static void main(String[] args) {
        final ReadWriteLockDemo rw = new ReadWriteLockDemo();
        // 写
        new Thread(new Runnable() {
            public void run() {
                rw.set((int)(Math.random()*100));
            }
        },"Write").start();
        // 读
        for (int i = 0; i < 100; i++) {
            new Thread(new Runnable() {
                public void run() {
                    rw.get();
                }
            },"Read").start();
        }

    }

}

class ReadWriteLockDemo {

    private int number = 0;

    private ReadWriteLock lock = new ReentrantReadWriteLock();

    /**
     * 读操作
     */
    public void get(){
        // 上锁
        lock.readLock().lock();
        try {
            System.out.println(Thread.currentThread().getName()+" : "+number);
        } finally {
            // 释放锁
            lock.readLock().unlock();
        }
    }

    /**
     * 写操作
     */
    public void set(int number){
        //上锁
        lock.writeLock().lock();
        try {
            System.out.println(Thread.currentThread().getName());
            this.number = number;
        } finally {
            // 释放锁
            lock.writeLock().unlock();
        }
    }

}
```

