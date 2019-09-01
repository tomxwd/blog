---
title: 01_Arrays.asList
date: 2019-09-01 11:16:47
tags: 
 - Java
 - 坑
categories:
 - 坑
---

# 01_Arrays.asList

## 1. 基本类型和包装类型使用asList的区别

```java
@Test
public void testArraysasList(){
    int[] arr1 = new int[]{1,2,3,4,5};
    Integer[] arr2 = new Integer[]{1,2,3,4,5};
    System.out.println(Arrays.asList(arr1).size());
    System.out.println(Arrays.asList(arr2).size());
}
```

输出结果：

```
1
5
```

为什么size不一样呢？查看该方法源码：

```java
public static <T> List<T> asList(T... a) {
    return new ArrayList<>(a);
}
```

发现asList接收的是一个泛型参数，再重新new一个ArrayList；而int是基本类型，基本类型不支持泛型化，但是数组支持，所以这时候传入的a是一个数组，因此size为1；



## 2. 对asList返回的ArrayList添加元素

```java
@Test
public void testArraysAsListAdd(){
    Integer[] arr = new Integer[]{1,2,3,4,5};
    List<Integer> list = Arrays.asList(arr);
    list.add(6);
}
```

结果居然报错了：

```
java.lang.UnsupportedOperationException
....
```

抛出`java.lang.UnsupportedOperationException`异常。

再次查看源码：

```java
private static class ArrayList<E> extends AbstractList<E>
        implements RandomAccess, java.io.Serializable
```

发现asList返回的ArrayList并不是`java.util.ArrayList`，而是`java.util.Arrays.ArrayList`类，这个类继承了一个**抽象类AbstractList**，但是在这个ArrayList类中并没有对add方法进行实现，所以asList返回的ArrayList并不支持修改操作。

**想要使用add或者remove可以通过类型转化，转化为ArrayList类即可**

```java
@Test
public void testArraysAsListAdd(){
    Integer[] arr = new Integer[]{1,2,3,4,5};
    List<Integer> list = Arrays.asList(arr);
    ArrayList<Integer> list1 = new ArrayList<Integer>(list);
    list1.add(6);
    System.out.println(list1);
}
```

