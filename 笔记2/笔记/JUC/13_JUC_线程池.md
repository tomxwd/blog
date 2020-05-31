---
2019-08-27 08:03:04

---

#

为什么需要线程池？

传统用法：

```java
/**
 * 线程池
 */
public class TestThreadPool {

    public static void main(String[] args) {
        new Thread(new ThreadPoolDemo(),"ThreadPoolDemo").start();
    }


}

class ThreadPoolDemo implements Runnable{

    private int i = 0;

    public void run() {
        while (i<=100){
            System.out.println(Thread.currentThread().getName()+":"+i++);
        }
    }
}
```

会带来频繁创建、销毁线程，非常耗费资源（类比数据库连接池）；



线程池：提供了一个线程队列，队列中保存着所有等待状态的线程，因此避免了创建与销毁额外开销，提高了响应速度。



线程池的体系结构：

java.util.concurrent.Executor：负责线程的使用与调度的根接口；

- ExecutorService 子接口：线程池的主要接口
  - ThreadPoolExecutor：线程池的实现类
  - ScheduledExecutorService：负责线程调度的子接口
    - ScheduledThreadPoolExecutor：继承了ThreadPoolExecutor并实现了ScheduledExecutorService；



Executors工具类：

ExecutorService newFixedThreadPool()：创建固定大小的线程池

ExecutorService newCachedThreadPool()：缓存线程池，线程池的数量不固定，可以根据需求自动的更改数量

ExecutorService newSingleThreadExecutor()：创建单个的线程池。线程池中只有一个线程



ScheduledExecutorService newScheduledThreadPool()：创建固定大小的线程池，还可以延迟或者定时的执行任务。



```java
/**
 * 线程池
 */
public class TestThreadPool {

    public static void main(String[] args) throws ExecutionException, InterruptedException {

        // 1. 创建线程池
        ExecutorService pool = Executors.newFixedThreadPool(5);
        List<Future<Integer>> list = new ArrayList<Future<Integer>>();
        for (int i=0;i<10;i++) {
            Future<Integer> future = pool.submit(new Callable<Integer>() {
                public Integer call() throws Exception {
                    int sum = 0;
                    for (int i = 0; i <= 100; i++) {
                        sum += i;
                    }
                    return sum;
                }
            });
            list.add(future);
        }

        for (Future<Integer> future : list) {
            System.out.println("future.get() = " + future.get());
        }

        ThreadPoolDemo tpd = new ThreadPoolDemo();
        // 2. 为线程池中的线程分配任务
        for (int i=0;i<10;i++) {
            pool.submit(tpd);
        }
        // 3. 关闭线程池
        // 平和的方式关闭
        pool.shutdown();
        // 暴力关闭
//        pool.shutdownNow();

    }


}

class ThreadPoolDemo implements Runnable{

    private int i = 0;

    public void run() {
        while (i<=100){
            System.out.println(Thread.currentThread().getName()+":"+i++);
        }
    }
}
```



