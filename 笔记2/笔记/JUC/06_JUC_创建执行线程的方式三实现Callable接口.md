---
2019-08-26 14:39:14

---

#

创建执行线程的方式一共有四种：

- 实现Runnable接口并重写run方法；
- 继承Thread类并重写run方法；
- 实现Callable接口
- 线程池

执行Callable方式，需要FutureTask实现类的支持，用于接收运算结果。

FutureTask是Future接口的实现类。

用FutureTask可以实现上一节的闭锁效果，即运算完了再执行get操作，达到效果；

示例代码：

```java
/**
 * 创建执行线程的方式3：实现Callable接口
 * 执行Callable方式，需要FutureTask实现类的支持，用于接收运算结果
 * FutureTask是Future接口的实现类
 */
public class TestCallable {

    public static void main(String[] args) {
        CallableDemo cd = new CallableDemo();
        FutureTask<Integer> result = new FutureTask<Integer>(cd);
        new Thread(result).start();
        try {
            Integer sum = result.get();
            System.out.println("sum = " + sum);
            System.out.println("-----------------------");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
    }

}

/**
 * 以往使用的实现Runnable接口
 */
class RunnableDemo implements Runnable{

    public void run() {

    }
}

/**
 * 跟Runnable接口相比，Callable是有返回值的，且有异常抛出
 */
class CallableDemo implements Callable<Integer>{

    public Integer call() throws Exception {
        int sum = 0;
        for (int i = 0; i < 100000; i++) {
            sum += i;
        }
        return sum;
    }

}
```

