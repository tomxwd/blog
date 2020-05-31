---
2019-09-04 14:41:44

---

#

##接口中的默认方法

Java8中允许接口中包含具有具体实现的方法，该方法称为“默认方法”，默认方法使用**default**关键字修饰。



###接口默认方法的“类优先”原则

若一个接口中定义了一个默认方法，而另外一个父类或接口中又定义了一个同名的方法时：

- 选择父类中的方法。

  如果一个父类提供了具体的实现，那么接口中具有相同名称和参数的默认方法会被忽略；

- 接口冲突。

  如果一个父接口提供一个默认方法，而另一个接口也提供了一个具有相同名称和参数列表的方法（不管方法是否默认方法），那么必须覆盖该方法来解决冲突。

####测试1：父类和接口相同方法

MyFun接口：

```java
public interface MyFun {

    default String getName(){
        return "哈哈哈";
    }

}
```

MyClass类：

```java
public class MyClass {

    public String getName(){
        return "嘿嘿";
    }

}
```

SubClass类：

```java
public class SubClass extends MyClass implements MyFun{
}
```

测试：

```java
public class TestDefaultInterface {

    public static void main(String[] args) {
        SubClass subClass = new SubClass();
        String name = subClass.getName();
        System.out.println("name = " + name);
    }

}
```

输出：

```
name = 嘿嘿
```

调用的是父类方法；

####测试2：接口冲突

MyFun2接口：

```java
public interface MyFun2 {

    default String getName(){
        return "呵呵";
    }

}
```

SubClass类：

```java
public class SubClass implements MyFun,MyFun2{
    @Override
    public String getName() {
        return MyFun2.super.getName();
    }
}
```

测试：

```java
public class TestDefaultInterface {

    public static void main(String[] args) {
        SubClass subClass = new SubClass();
        String name = subClass.getName();
        System.out.println("name = " + name);
    }

}
```

**注意到interface的super调用getName方法。**



##接口中的静态方法

接口MyFun3：

```java
public interface MyFun3 {

    static void show(){
        System.out.println("接口静态方法");
    }

}
```

测试：

```java
@Test
public void test(){
    MyFun3.show();
}
```

一样的调用方式；