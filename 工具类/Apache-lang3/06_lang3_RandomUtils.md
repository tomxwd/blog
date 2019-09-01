---
title: 06_lang3_RandomUtils
date: 2019-07-30 11:30:19
tags: 
 - Java
 - Apache-lang3
categories:
 - Apache-lang3
---

# 06_lang3_RandomUtils

## 返回基本数据类型随机数

左闭右开[x，y)

```java
@Test
public void testRandomData (){
    // 返回一个随机布尔值,还有其他基本数据类型返回随机值
    boolean b = RandomUtils.nextBoolean();
    System.out.println("b = " + b);
    int i = RandomUtils.nextInt();
    System.out.println("i = " + i);
    int i1 = RandomUtils.nextInt(10, 20);
    System.out.println("i1 = " + i1);
    byte[] bytes = RandomUtils.nextBytes(2);
    System.out.println(Arrays.toString(bytes));
    double v = RandomUtils.nextDouble(1.0, 90.0);
    System.out.println("v = " + v);
}
```

结果：

```
b = true
i = 1468851514
i1 = 13
[47, -85]
v = 5.816467547986196
```



