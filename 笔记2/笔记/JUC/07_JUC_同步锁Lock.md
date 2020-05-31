---
2019-08-26 15:04:09

---

#

Lock同步锁

用于解决多线程安全问题的方式：

1. 同步代码块(synchronized)【隐式锁】
2. 同步方法(synchronized)【隐式锁】
3. jdk1.5以后出现了一种更加灵活的方式：同步锁Lock【显式锁】，需要通过lock()方法进行上锁，相应的必须通过unlock()方法进行释放锁

```java
public class TestLock {

    public static void main(String[] args) {
        Ticket ticket = new Ticket();
        new Thread(ticket,"1号窗口").start();
        new Thread(ticket,"2号窗口").start();
        new Thread(ticket,"3号窗口").start();
    }

}

class Ticket implements Runnable{
    private int tick = 100;

    private Lock lock = new ReentrantLock();

    private void ru(){
        while (tick>0){
            synchronized (this){
                try {
                    Thread.sleep(20);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName()+" ： 完成售票，余票为"+(--tick));
            }
        }
    }

    public void run() {
//        ru();
        while (true){
            // 记住要先锁再判断tick是否>0，否则在while条件里判断的话，可能会导致出现负数
            lock.lock();
            try {
                if (tick>0) {
                    try {
                        Thread.sleep(20);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println(Thread.currentThread().getName()+" ： 完成售票，余票为"+(--tick));
                }
            } finally {
                lock.unlock();
            }
        }
    }
}
```

