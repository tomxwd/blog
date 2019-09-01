---
title: 03_JUC_模拟 CAS 算法
date: 2019-08-26 11:23:36
tags: 
 - Java
 - JUC
categories:
 - JUC
---

# 03_JUC_模拟 CAS 算法

## CAS算法

- CAS（Compare-And-Swap）是一种硬件对并发的支持，针对多处理器操作而设计的处理器中的一种特殊指令，用于管理对共享数据的并发访问；
- CAS是一种无锁的非阻塞算法的实现；
- CAS包含了3个操作数：
  - 需要读写的内存值V
  - 进行比较的值A
  - 拟写入的新值B
- 当且仅当V的值等于A时，CAS通过原子方式用新值B来更新V的值，否则不会执行任何操作；



## 模拟CAS算法

```java
/**
 * 模拟CAS算法
 */
public class TestCompareAndSwap {

    public static void main(String[] args) {
        final CompareAndSwap cas = new CompareAndSwap();
        for (int i = 0; i < 10; i++) {
            new Thread(new Runnable() {
                public void run() {
                    int expectedValue = cas.getValue();
                    int newValue = (int)(Math.random()*100);
                    boolean b = cas.compareAndSet(expectedValue, newValue);
                    System.out.println(Thread.currentThread().getName()+" : "+b);
                }
            }).start();
        }
    }

}

class CompareAndSwap{
    private int value;

    /**
     * 获取内存值
     * @return
     */
    public synchronized int getValue(){
        return value;
    }

    /**
     * 比较
     * @param expectedValue
     * @param newValue
     * @return
     */
    public synchronized int comparedAndSwap(int expectedValue,int newValue){
        int oldValue = value;
        if(oldValue == expectedValue){
            this.value = newValue;
        }
        return oldValue;
    }

    /**
     * 设置
     * @param expectedValue
     * @param newValue
     * @return
     */
    public synchronized boolean compareAndSet(int expectedValue,int newValue){
        return expectedValue == comparedAndSwap(expectedValue,newValue);
    }

}
```

输出：

```
Thread-0 : true
Thread-1 : false
Thread-3 : false
Thread-5 : true
Thread-4 : true
Thread-2 : false
Thread-8 : true
Thread-7 : true
Thread-6 : true
Thread-9 : true
```

