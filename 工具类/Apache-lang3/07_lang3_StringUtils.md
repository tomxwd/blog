---
title: 07_lang3_StringUtils
date: 2019-07-30 14:24:16
tags: 
 - Java
 - Apache-lang3
categories:
 - Apache-lang3
---

# 07_lang3_StringUtils

## abbreviate仅显示固定位数字符串，多余的用...代替

```java
@Test
public void testAbbreviate(){
    // 只显示5位，且如果后面5位，则后三位是省略号
    // maxWidth必须大于3，否则不够放省略号报错 java.lang.IllegalArgumentException: Minimum abbreviation width is 4
    String str = StringUtils.abbreviate("1231231234", 5);
    System.out.println("str = " + str);
}
```

结果：

```
str = 12...
```



## abbreviateMiddle仅显示固定位数，且中间部分用连接符

```java
@Test
public void testAbbreviateMiddle(){
    // 只显示8位，中间用--连接
    String str = StringUtils.abbreviateMiddle("ABCDEFGHIJK", "--", 8);
    System.out.println("str = " + str);
}
```

结果：

```
str = ABC--IJK
```



## appendIfMissing拼接字符串，存在重复则不拼接

```java
@Test
public void testAbbreviateMiddle(){
    // 只显示8位，中间用--连接
    String str = StringUtils.abbreviateMiddle("ABCDEFGHIJK", "--", 8);
    System.out.println("str = " + str);
}
```

结果：

```
str4 = 123ABCDEF
str5 = 123ABC
```



## center补充字符，以字符为中心

```java
@Test
public void testCenter(){
    // 补充字符，如果不到size，就用"*"补充
    String str = StringUtils.center("xwd", 20,"*");
    System.out.println("str = " + str);
}
```

结果：

```
str = ********xwd*********
```



## contains是否包含

```java
@Test
public void testContains(){
    // 是否包含字符串
    String str = "tomxwdtomatomtomttttomtom";
    boolean b = StringUtils.contains(str, "toma");
    System.out.println("b = " + b);
}
```

结果：

```
b = true
```



## containsAny是否包含任意一项

```java
@Test
public void testContainsAny(){
    // 是否包含任意一个字符串
    String str = "tttomxwdttiixxwdiiwwwmmm";
    boolean b = StringUtils.containsAny(str, "tttt", "to4m", "xwd");
    System.out.println("b = " + b);
}
```

结果：

```
b = true
```



## defaultString，null和空字符串转为空字符串，若有内容，则返回内容

```java
@Test
public void testDefaultString(){
    // null和空字符串转为空字符串，若有内容，则返回内容
    String str1 = null;
    String str2 = "";
    String str3 = "X";
    String ds1 = StringUtils.defaultString(str1);
    String ds2 = StringUtils.defaultString(str2);
    String ds3 = StringUtils.defaultString(str3);
    System.out.println("ds1 = " + ds1);
    System.out.println("ds2 = " + ds2);
    System.out.println("ds3 = " + ds3);
}
```

结果：

```
ds1 = 
ds2 = 
ds3 = X
```



## defaultIfBlank若字符串为null或空或空白，则提供默认字符替代

```java
@Test
public void testDefaultBlank(){
    // 如果是空或者null，返回默认一个字符串
    String str1 = null;
    String str2 = "";
    String str3 = "  ";
    String dibs1 = StringUtils.defaultIfBlank(str1, "ttt");
    String dibs2 = StringUtils.defaultIfBlank(str2, "tt1t");
    String dibs3 = StringUtils.defaultIfBlank(str3, "554");
    System.out.println("dibs1 = " + dibs1);
    System.out.println("dibs2 = " + dibs2);
    System.out.println("dibs3 = " + dibs3);
}
```

结果：

```
dibs1 = ttt
dibs2 = tt1t
dibs3 = 554
```



## endsWith是否以字符为结尾

```java
@Test
public void testEndWith(){
    // 是否以endStr结尾
    String str1 = "ttxxwwff";
    boolean b = StringUtils.endsWith(str1,"ff");
    System.out.println("b = " + b);
}
```

结果：

```
b = true
```



## equals是否相同

```java
@Test
public void testEquals(){
    // 字符串是否相同
    String str1 = "xx";
    String str2 = "xx";
    boolean equals = StringUtils.equals(str1, str2);
    System.out.println("equals = " + equals);
}
```

结果：

```
equals = true
```



## isEmpty是否为null或空

```java
@Test
public void testIsEmpty(){
    // 检测是否为空或null
    String str1 = null;
    String str2 = "";
    String str3 = "    ";
    boolean b1 = StringUtils.isEmpty(str1);
    boolean b2 = StringUtils.isEmpty(str2);
    boolean b3 = StringUtils.isEmpty(str3);
    System.out.println("b1 = " + b1);
    System.out.println("b2 = " + b2);
    System.out.println("b3 = " + b3);
}
```

结果：

```
b1 = true
b2 = true
b3 = false
```



## isBlank是否为null或空或空白字符串

```java
@Test
public void testIsBlank(){
    // 检测是否为 空白字符串 或 为null
    String str1 = null;
    String str2 = "";
    String str3 = "    ";
    boolean b1 = StringUtils.isBlank(str1);
    boolean b2 = StringUtils.isBlank(str2);
    boolean b3 = StringUtils.isBlank(str3);
    System.out.println("b1 = " + b1);
    System.out.println("b2 = " + b2);
    System.out.println("b3 = " + b3);
}
```

结果：

```
b1 = true
b2 = true
b3 = true
```



## join连接字符串，用什么间隔，可指定范围

```java
@Test
public void testJoin(){
    // 数组转字符连接
    Integer[] integers = new Integer[]{1,2,3,4,5,6,7,8,9,10};
    Dog[] dogs = new Dog[]{new Dog("D1"),new Dog("D2"),new Dog("D3"),new Dog("D4")};
    String str1 = StringUtils.join(integers,"-");
    String str2 = StringUtils.join(integers,"--",2,5);
    String str3 = StringUtils.join(dogs,"+");
    String str4 = StringUtils.join(dogs,"++",0,2);
    System.out.println("str1 = " + str1);
    System.out.println("str2 = " + str2);
    System.out.println("str3 = " + str3);
    System.out.println("str4 = " + str4);
}
```

结果：

```
str1 = 1-2-3-4-5-6-7-8-9-10
str2 = 3--4--5
str3 = Dog(name=D1)+Dog(name=D2)+Dog(name=D3)+Dog(name=D4)
str4 = Dog(name=D1)++Dog(name=D2)
```



## repeat重复字符串，可指定次数以及分隔符

```java
@Test
public void testRepeat(){
    // 重复的字符串控制在指定大小,可指定分隔符
    String str = "tttttxxxxxwwwwww";
    String x = StringUtils.repeat(str, "-", 2);
    System.out.println("x = " + x);
}
```

结果：

```
x = tttttxxxxxwwwwww-tttttxxxxxwwwwww
```



## replace替换字符串，可指定替换次数

```java
@Test
public void testReplace(){
    // 替换字符串,且可以指定替换次数
    String str1 = "x22xw23d";
    String str2 = StringUtils.replace(str1, "2", "T", 2);
    System.out.println("str1 = " + str1);
    System.out.println("str2 = " + str2);
}
```

结果：

```
str1 = x22xw23d
str2 = xTTxw23d
```



## reverse逆序

```java
@Test
public void testReverse(){
    // 逆序
    String str1 = "dwxmot";
    String str2 = StringUtils.reverse(str1);
    System.out.println("str1 = " + str1);
    System.out.println("str2 = " + str2);
}
```

结果：

```
str1 = dwxmot
str2 = tomxwd
```

