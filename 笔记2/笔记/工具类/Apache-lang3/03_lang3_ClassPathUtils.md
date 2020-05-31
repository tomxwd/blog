---
2019-07-29 17:38:14

---





## 1. toFullyQualifiedName获取指定类全限定名

可以自定义第二个参数。

```java
@Test
public void TestFullyQualifiedName(){
    String str = ClassPathUtils.toFullyQualifiedName(Lang3ClasspathUtilsTest.class, "Lang3Test");
    System.out.println("str = " + str);
}
```

结果：

```
str = top.tomxwd.other.lang3.Lang3Test
```





## 2. toFullyQualifiedPath获取指定类路径

```java
@Test
public void TestFullyQualifiedPath(){
    String str = ClassPathUtils.toFullyQualifiedPath(Lang3ClasspathUtilsTest.class,"lang3Test");
    System.out.println("str = " + str);
}
```

结果：

```
str = top/tomxwd/other/lang3/lang3Test
```

