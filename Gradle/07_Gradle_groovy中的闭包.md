---
title: 07_Gradle_groovy中的闭包
date: 2019-08-27 14:14:34
tags: 
 - gradle
categories:
 - Java
 - gradle
---

# 07_Gradle_groovy中的闭包
groovy中的闭包

什么是闭包？

闭包就是一段代码块，在gradle中，我们主要是把闭包当参数来使用。

1. 首先定义一个闭包
2. 定义一个方法，参数为闭包类型
3. 调用该方法

简单闭包（不带参数）：

```groovy
// 定义一个闭包
def b1 = {
    println "hello b1"
}
// 定义一个方法，方法里面需要闭包类型的参数
def method1(Closure closure){
    closure()
}
// 调用方法method1
method1(b1)
```

带传入参数的闭包：

```groovy
// 定义一个闭包（带参数）
def b2 = {
    v ->
        println "hello ${v}"
}
// 定义一个方法，方法里面需要闭包类型的参数
def method2(Closure closure){
    closure("xxx")
}
method2(b2)
```

