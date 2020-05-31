---
2019-07-30 16:14:12

---



Java虚拟机栈

虚拟机栈描述的是Java方法执行的动态内存模型。

1. 栈帧：
   - 每个方法执行，都会创建一个栈帧，伴随着方法从创建到执行完成。
   - 用于存储局部变量表，操作数栈，动态链接，方法出口等。
2. 局部变量表：
   - 存放编译期可知的各种基本数据类型，引用类型，returnAddress类型
   - 局部变量表的内存空间在编译期就完成分配，当进入一个方法时，这个方法需要在栈中分配多少内存是固定的，在方法运行期间是不会改变局部变量表的大小。
3. 大小：
   - StackOverflowError：栈满
   - OutOfMemoryError（OOM）：内存溢出



如果栈满了，就会抛出异常**StackOverflowError**；写一个递归的时候就十分常见。

模拟：

```java
public class MyTest24 {

    @Test
    public void stackOverFolwErrorTest(){
        exec();
    }

    public void exec(){
        System.out.println("方法执行");
        exec();
    }

}
```

结果会程序运行中断，并抛出**java.lang.StackOverflowError**异常。