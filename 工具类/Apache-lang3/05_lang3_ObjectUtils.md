---
title: 05_lang3_ObjectUtils
date: 2019-07-30 10:57:57
tags: 
 - Java
 - Apache-lang3
categories:
 - Apache-lang3
---

# 05_lang3_ObjectUtils

## allNotNull判断是否全不为空

```java
@Test
public void testAllNotNull(){
    // 全部不为空
    boolean b = ObjectUtils.allNotNull(new Student(),null);
    System.out.println("b = " + b);
}
```

结果：

```
b = false
```



## anyNotNull判断是否至少有一个不为kong

```java
@Test
public void testAnyNotNull(){
    // 至少有一个不为空
    boolean b = ObjectUtils.anyNotNull("", null, new Student(), null);
    System.out.println("b = " + b);
}
```

结果：

```
b = true
```



## firstNonNull返回第一个不为空的对象

```java
@Test
public void testFirstNotNull(){
    // 返回第一个非空对象 要实现Cloneable
    Cloneable cloneable = ObjectUtils.firstNonNull(null, new Dog(), new Student());
    System.out.println(cloneable);
    Dog dog = (Dog) ObjectUtils.firstNonNull(null, new Dog(), new Student());
    System.out.println(dog);
}
```

结果：

```
Dog(name=null)
Dog(name=null)
```



## clone克隆对象

```java
@Test
public void testClone(){
    // 需要实现Cloneable 否则空指针异常
    Student s = new Student(1, "S", new Dog("D"));
    Student clone = ObjectUtils.clone(s);
    clone.getDog().setName("123");
    System.out.println("s = " + s);
    System.out.println("clone = " + clone);
}
```

结果：

```
s = Student(no=1, name=S, dog=Dog(name=D))
clone = Student(no=1, name=S, dog=Dog(name=123))
```

