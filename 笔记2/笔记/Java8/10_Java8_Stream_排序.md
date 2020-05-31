---
2019-09-03 13:46:51

---

#

##排序

| 方法                    | 描述                                                 |
| ----------------------- | ---------------------------------------------------- |
| sorted()                | 产生一个新流，其中按**自然顺序排序（Comparable）**   |
| sorted(Comparator comp) | 产生一个新流，其中按**比较器顺序排序（Comparator）** |

```java
@Test
public void test8(){
    // 字符排序（自然排序）
    List<String> list = Arrays.asList("ddd","bbb","eee","aaa","ccc");
    list.stream()
        .sorted()
        .forEach(System.out::println);
    System.out.println("----------------------------");
    // 年龄排序（定制排序）
    employees.stream()
        .sorted((o1, o2) -> o1.getAge().compareTo(o2.getAge()))
        .forEach(System.out::println);
}
```

