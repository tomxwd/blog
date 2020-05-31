---
2019-07-30 10:57:02

---



## 字符串是否在枚举中

```java
@Test
public void test(){
    boolean validEnum = EnumUtils.isValidEnum(Type.class, "12");
    System.out.println("validEnum = " + validEnum);
}
```

