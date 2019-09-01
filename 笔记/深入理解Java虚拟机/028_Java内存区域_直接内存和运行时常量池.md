---
2019-08-02 15:09:36
---











运行时常量池（在方法区中）

- 存放编译期生成的各种字面量以及符号引用，这部分内容将在类加载后进入方法区的运行池常量池中存放



测试：

```java
@Test
public void testString(){

    String s1 = "abc";
    String s2 = "abc";

    System.out.println("s1==s2 = " + (s1 == s2));
    
    String s3 = new String("abc");

    System.out.println("s1==s3 = " + (s1 == s3));
    
    System.out.println("s1==s3 = " + (s1 == s3.intern()));

}
```

结果：

```
s1==s2 = true
s1==s3 = false
s1==s3 = true
```



可以想象成在方法区中有个运行时常量池，然后池里有个StringTable，类型是HashSet，存放实例化的"abc"对象，又由于HashSet是**无序不可重复**的，所以"abc"和"abc"显然是相等的。

而new出来的对象，是在运行的时候才分配内存的，所以不会在运行时常量池中。因此是不相等的。



String.intern()方法：

既是将堆中的"str"移到运行池常量池中。

String s1 = "abc"这种在编译期就确定的，叫字节码常量；

s3.intern()称之为运行时常量；



直接内存：

- **分配堆外内存**，不受制于java虚拟机内存的制约，但是也会受到物理机的内存制约。