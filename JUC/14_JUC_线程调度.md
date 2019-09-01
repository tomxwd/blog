---
title: 14_JUC_线程调度
date: 2019-08-27 08:54:26
tags: 
 - Java
 - JUC
categories:
 - JUC
---

# 14_JUC_线程调度

ScheduledExecutorService

```java
public class TestScheduledThreadPool {

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        ScheduledExecutorService pool = Executors.newScheduledThreadPool(5);

        for (int i=0;i<5;i++) {
            Future<Integer> result = pool.schedule(new Callable<Integer>() {
                public Integer call() throws Exception {
                    int num = new Random().nextInt(100);
                    System.out.println(Thread.currentThread().getName() + ":" + num);
                    return num;
                }
            }, 1, TimeUnit.SECONDS);
            System.out.println("result.get() = " + result.get());
        }
        pool.shutdown();

    }

}
```



