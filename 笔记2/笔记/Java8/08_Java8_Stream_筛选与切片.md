---
2019-09-03 10:11:37

---

#



##Stream的中间操作

多个**中间操作**可以连接起来形成一个**流水线**，除非流水线上触发终止操作，否则**中间操作不会执行任何的处理**！而在**终止操作时一次性全部处理，称为“惰性求值”；

准备工作：

```java
List<Employee> employees = Arrays.asList(
    new Employee("张三",18,999.3),
    new Employee("李四",38,5555.99),
    new Employee("王五",17,6636.66),
    new Employee("赵六",55,333.3),
    new Employee("田七",13,778.3)
);
```



###筛选与切片

| 方法                | 描述                                                         |
| ------------------- | ------------------------------------------------------------ |
| filter(Predicate p) | 接收Lambda，从流中排除某些元素                               |
| distinct()          | 筛选，通过流所生成元素的hashCode()和equals()去除重复元素     |
| limit(long maxSize) | 截断流，使其元素不超过给定数量                               |
| skip(long n)        | 跳过元素，返回一个扔掉了前n个元素的流。若流中元素不足n个，则返回一个空流。与limit(n)互补。 |

- filter

  ```java
  // 内部迭代：迭代操作由Stream API完成
  @Test
  public void test1(){
      // filter:接收Lambda，从流中排除某些元素
      // 中间操作不会执行任何操作
      Stream<Employee> stream = employees.stream()
          .filter((e) -> {
              System.out.println("中间操作");
              return e.getAge() > 35;
          });
      // 终止操作：一次性执行全部内容，即“惰性求值”
      stream.forEach(System.out::println);
  }
  
  // 外部迭代：
  @Test
  public void test2(){
      Iterator<Employee> it = employees.iterator();
      while (it.hasNext()){
          System.out.println(it.next());
      }
  }
  ```

  通过对代码进行断点调试，可以知道中间操作并不会直接执行，而是到终止操作才会执行，也就是“惰性求值”；

- distinct

  先在成员变量employees上暂时添加一条，作为测试需求，即两条同样的数据即可；

  **注意：**这里要对Employee对象重写hashCode和equals方法才可以

  ```java
  @Test
  public void test5(){
      employees.stream()
          .distinct().forEach(System.out::println);
  }
  ```

- limit

  ```java
  @Test
  public void test3(){
      employees.stream()
          .filter(e->{
              System.out.println("短路");
              return e.getSalary()>900;
          })
          .limit(2)
          .forEach(System.out::println);
  }
  ```

  查看控制台输出可以知道，只进行了迭代4次，那么就意味着，只要找到两个就会结束迭代，这种叫做**短路**；

- skip

  ```java
  @Test
  public void test4(){
      employees.stream()
          .filter(e->e.getSalary()>900)
          .skip(2)
          .forEach(System.out::println);
  }
  ```

  



