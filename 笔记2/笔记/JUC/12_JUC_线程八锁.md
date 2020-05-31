---
2019-08-26 17:21:15

---

#

## 

两个普通同步方法，两个线程标准打印

代码：

```java
public class TestThread8Monitor extends Thread {

    public static void main(String[] args) {
        final Number number = new Number();
        new Thread(new Runnable() {
            public void run() {
                number.getOne();
            }
        }).start();
        new Thread(new Runnable() {
            public void run() {
                number.getTwo();
            }
        }).start();
    }

}

class Number{

    public synchronized void getOne(){
        System.out.println("one");
    }

    public synchronized void getTwo(){
        System.out.println("two");
    }

}
```

输出：

```
one
two
```



##

getOne()休眠3秒，看输出

代码：

```java
public class TestThread8Monitor extends Thread {

    public static void main(String[] args) {
        final Number number = new Number();
        new Thread(new Runnable() {
            public void run() {
                number.getOne();
            }
        }).start();
        new Thread(new Runnable() {
            public void run() {
                number.getTwo();
            }
        }).start();
    }

}

class Number{

    public synchronized void getOne(){
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("one");
    }

    public synchronized void getTwo(){
        System.out.println("two");
    }

}
```

输出：

三秒后输出

```
one
two
```



##

定义一个新的普通方法，输出three；

代码：

```java
public class TestThread8Monitor extends Thread {

    public static void main(String[] args) {
        final Number number = new Number();
        new Thread(new Runnable() {
            public void run() {
                number.getOne();
            }
        }).start();
        new Thread(new Runnable() {
            public void run() {
                number.getTwo();
            }
        }).start();
        new Thread(new Runnable() {
            public void run() {
                number.getThree();
            }
        }).start();
    }

}

class Number{

    public synchronized void getOne(){
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("one");
    }

    public synchronized void getTwo(){
        System.out.println("two");
    }

    public void getThree(){
        System.out.println("three");
    }

}
```

输出：

先直接输出three，大概三秒后输出one和two；

```
three
one
two
```



##

把getOne()定义为静态同步方法；

代码：

```java
public class TestThread8Monitor extends Thread {

    public static void main(String[] args) {
        final Number number = new Number();
        new Thread(new Runnable() {
            public void run() {
                number.getOne();
            }
        }).start();
        new Thread(new Runnable() {
            public void run() {
                number.getTwo();
            }
        }).start();
    }

}

class Number{

    public static synchronized void getOne(){
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("one");
    }

    public synchronized void getTwo(){
        System.out.println("two");
    }

}
```

输出：

先输出two，三秒后输出one

```
two
one
```



##

两个都声明为静态同步方法；

代码：

```java
public class TestThread8Monitor extends Thread {

    public static void main(String[] args) {
        final Number number = new Number();
        new Thread(new Runnable() {
            public void run() {
                number.getOne();
            }
        }).start();
        new Thread(new Runnable() {
            public void run() {
                number.getTwo();
            }
        }).start();
    }

}

class Number{

    public static synchronized void getOne(){
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("one");
    }

    public static synchronized void getTwo(){
        System.out.println("two");
    }

}
```

输出：

三秒后输出one，two；

```
one
two
```



##

new两个Number对象，一个调用getOne()方法，一个调用getTwo()方法；

代码：

```java
public class TestThread8Monitor extends Thread {

    public static void main(String[] args) {
        final Number number = new Number();
        final Number number2 = new Number();
        new Thread(new Runnable() {
            public void run() {
                number.getOne();
            }
        }).start();
        new Thread(new Runnable() {
            public void run() {
                number2.getTwo();
            }
        }).start();
    }

}

class Number{

    public synchronized void getOne(){
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("one");
    }

    public synchronized void getTwo(){
        System.out.println("two");
    }

}
```

输出：

直接输出two，三秒后输出one；

```
two
one
```



##

两个Number对象，getOne()为静态同步方法，getTwo为普通同步方法；

代码：

```java
public class TestThread8Monitor extends Thread {

    public static void main(String[] args) {
        final Number number = new Number();
        final Number number2 = new Number();
        new Thread(new Runnable() {
            public void run() {
                number.getOne();
            }
        }).start();
        new Thread(new Runnable() {
            public void run() {
                number2.getTwo();
            }
        }).start();
    }

}

class Number{

    public static synchronized void getOne(){
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("one");
    }

    public synchronized void getTwo(){
        System.out.println("two");
    }

}
```

输出：

先输出two，三秒后输出one

```
two
one
```



##

两个Number对象，两个方法均为静态同步方法

代码：

```java
public class TestThread8Monitor extends Thread {

    public static void main(String[] args) {
        final Number number = new Number();
        final Number number2 = new Number();
        new Thread(new Runnable() {
            public void run() {
                number.getOne();
            }
        }).start();
        new Thread(new Runnable() {
            public void run() {
                number2.getTwo();
            }
        }).start();
    }

}

class Number{

    public static synchronized void getOne(){
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("one");
    }

    public static synchronized void getTwo(){
        System.out.println("two");
    }

}
```

输出：

三秒后输出one，two

```
one
two
```



##

1. 非静态方法的锁默认为this，静态方法的锁为对应的Class实例
2. 某一个时刻内，只能有一个线程持有锁，无论几个方法