---
2019-09-02 16:13:33

---

#

##

操作符

Lambda表达式的基础语法：Java8中引入了一个新的操作符"->"，该操作符成为**箭头操作符** 或者是**Lambda操作符**；

箭头操作符将Lambda表达式拆分为两个部分：

- 左侧：Lambda表达式的参数列表
- 右侧：Lambda表达式中所需要执行的功能，即Lambda体



##

语法格式

1. 无参数，无返回值：()->System.out.println("Hello Lambda")；

   ```java
   @Test
   public void test1(){
       Runnable r = ()->System.out.println("Hello Lambda");
   }
   ```

2. 一个参数，无返回值：

   ```java
   @Test
   public void test2(){
       Consumer<String> consumer = (x)-> System.out.println(x);
       consumer.accept("我");
   }
   ```

3. 一个参数，无返回值，那么小括号可以省略不写：

   ```java
   @Test
   public void test2(){
       Consumer<String> consumer = x-> System.out.println(x);
       consumer.accept("我");
   }
   ```

4. 两个以上参数，有返回值，并且Lambda体中有多条语句：

   ```java
   @Test
   public void test3(){
       Comparator<Integer> com = (x,y) -> {
           System.out.println("函数式接口");
           return Integer.compare(x,y);
       };
       System.out.println("com.compare(1, 2) = " + com.compare(1, 2));
   }
   ```

5. 两个以上参数，有返回值，Lambda体中只有一条语句：return和大括号都可以省略不写：

   ```java
   @Test
   public void test4(){
       Comparator<Integer> com = (x,y) -> Integer.compare(x,y);
       System.out.println("com.compare(1,2) = " + com.compare(1, 2));
   }
   ```

6. Lambda 表达式的参数列表的数据类型可以省略不写，因为JVM编译器可以通过上下文，推断出数据类型，即**类型推断**，如果要写，就要所有参数的类型都写上；

   - 类型推断：

     ```java
     @Test
     public void test5(){
         String[] strs = {"aaa","bbb","ccc"};
     }
     ```

     上面也是类型推断，但是当我们把声明和定义拆开两步，编译器就会报错，因为无法推断。

     ```java
     @Test
     public void test5(){
         List<String> list = new ArrayList<>();
     }
     ```

     这里后面的尖括号为什么不用写，也是因为类型推断；

   - JDK8升级类型推断：

     在JDK8中，类型推断进行了升级，如：

     ```java
     @Test
     public void test5(){
         show(new HashMap<>());
     }
     
     public void show(Map<String,Integer> map){
     
     }
     ```

     可以看到，在new的时候不写尖括号里的内容都可以，这样的代码如果换到JDK7中，会编译不通过；

     

###格式总结

能省则省，

左右遇一括号省，

左侧推断类型省。



##函数式接口

Lambda表达式需要**“函数式接口”**的支持；

函数式接口：若接口中只有一个抽象方法的接口，称为**函数式接口**；

可以使用注解**@FunctionalInterface**修饰，可以检查是否是函数式接口；

```java
@FunctionalInterface
public interface MyPredicate<T> {

    boolean test(T t);

}
```

如果定义两个抽象方法，就会报错；



##使用Lambda

准备好一个函数式接口：

```java
@FunctionalInterface
public interface MyFun {

    Integer getValue(Integer num);

}
```

测试：

```java
// 需求：对一个数进行运算
@Test
public void test6(){
    // 平方运算
    Integer num = operation(100, (x) -> x * x);
    System.out.println("num = " + num);
    // 对数+200运算
    num = operation(100, (x) -> x + 200);
    System.out.println("num = " + num);
}

public Integer operation(Integer num,MyFun mf){
    return mf.getValue(num);
}
```

