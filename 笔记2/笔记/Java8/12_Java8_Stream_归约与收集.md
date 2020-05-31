---
2019-09-03 14:30:45

---

#
##归约与收集

###归约

| 方法                                 | 描述                                                   |
| ------------------------------------ | ------------------------------------------------------ |
| reduce(T identity, BinaryOperator b) | 将流中的元素反复结合起来，得到一个值，返回T            |
| reduce(BinaryOperator b)             | 将流中的元素反复结合起来，得到一个值，返回Optional\<T> |

备注：map和reduce的连接通常称为map-reduce模式，因Google用它来进行网络搜索而出名。

```java
@Test
public void test3(){
    List<Integer> list = Arrays.asList(1,2,3,4,5,6,7,8,9,10);
    Integer sum = list.stream()
            .reduce(0, (x, y) -> x + y);
    System.out.println("sum = " + sum);
    System.out.println("--------------------------------");
    // 得到公司工资的总和
    Optional<Double> salarySum = employees.stream()
            .map(Employee::getSalary)
            .reduce(Double::sum);
    System.out.println("salarySum.get() = " + salarySum.get());
}
```



###收集

| 方法                 | 描述                                                         |
| -------------------- | ------------------------------------------------------------ |
| collect(Collector c) | 将流转换为其他形式。接收一个Collector接口的实现，用于给Stream中元素做汇总的方法。 |

Collector接口中方法的实现决定了如何对流执行收集操作（如收集到List、Set、Map）。但是**Collectors实用类**提供了很多静态方法，可以方便地创建常见收集器实例，具体方法与实例如下表：

| 方法              | 返回类型               | 用法                                                         | 作用                                                         |
| ----------------- | ---------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| toList            | List\<T>               | List\<Employee> emps = list.stream().collect(Collectors.toList()); | 把流中的元素收集到List                                       |
| toSet             | Set\<T>                | Set<Employee\> emps = list.stream().collect(Collectors.toSet()); | 把流中的元素收集到Set                                        |
| toCollection      | Collection\<T>         | Collection<Employee\> emps = list.stream().collect(Collectors.toCollection(ArrayList::new)); | 把流中元素收集到创建的集合                                   |
| counting          | Long                   | long count = list.stream().collect(Collectors.counting());   | 计算流中元素个数                                             |
| summingInt        | Integer                | int total = list.stream().collect(Collectors.summingInt(Employee::getSalary)); | 对流中元素的整数属性求和                                     |
| averagingInt      | Double                 | double avg = list.stream().collect(Collectors.averagingInt(Emoloyee::getSalary)); | 计算流中元素Integer属性的平均值                              |
| summarizingInt    | IntSummaryStatistics   | IntSummaryStatistics = list.stream().collect(Collectors.summarizingInt(Employee::getSalary)); | 收集流中Integer属性的统计值（包括平均值，求和等）            |
| joining           | String                 | String str = list.stream().map(Employee::getName).collect(Collectors.joining()); | 连接流中每个字符串                                           |
| maxBy             | Optional\<T>           | Optional\<Emp> max = list.stream().collect(Collectors.maxBy(comparingInt(Employee::getSalary))); | 根据比较器选择最大值                                         |
| minBy             | Optional\<T>           | Optional\<Emp> min = list.stream().collect(Collectors.minBy(comparingInt(Employee::getSalary))); | 根据比较器选择最小值                                         |
| reducing          | 归约产生的类型         | int total = list.stream().collect(Collectors.reducing(0,Employee::getSalary,Intger::sum)); | 从一个作为累加器的初始值开始，利用BinaryOperator与流中元素逐个结合，从而归约成单个值 |
| collectingAndThen | 转换函数返回的类型     | int how = list.stream().collect(Collectors.collectingAndThen(Collectors.toList(),List::size)); | 包裹另一个收集器，对其结果转换函数                           |
| groupingBy        | Map\<K,List\<T>>       | Map\<Emp.Status,List\<Emp>> map = list.stream().collect(Collectors.groupingBy(Employee::getStatus)); | 根据某个属性值对流分组，属性为K，结果为V                     |
| partitioningBy    | Map\<Boolean,List\<T>> | Map\<Boolean,List\<Emp>> vd = list.stream().collect(Collectors.partitioningBy(Employee::getAge)); | 根据true或false进行分区                                      |

转容器：

```java
@Test
public void test4(){
    // 把公司中所有员工的名字收集起来放入到list中
    List<String> names = employees.stream()
            .map(Employee::getName)
            .collect(Collectors.toList());
    System.out.println("names = " + names);
    System.out.println("----------------------");
    Set<String> set = employees.stream()
            .map(Employee::getName)
            .collect(Collectors.toSet());
    System.out.println("set = " + set);
    System.out.println("----------------------");
    HashSet<String> hashSet = employees.stream()
            .map(Employee::getName)
            .collect(Collectors.toCollection(HashSet::new));
    System.out.println("hashSet = " + hashSet);
}
```



求值：

```java
@Test
public void test5(){
    // 总数
    Long collect = employees.stream()
        .collect(Collectors.counting());
    System.out.println("collect = " + collect);
    System.out.println("------------------------");
    // 平均值
    Double averag = employees.stream()
        .collect(Collectors.averagingDouble(Employee::getSalary));
    System.out.println("averag = " + averag);
    System.out.println("------------------------");
    // 总和
    Double sum = employees.stream()
        .collect(Collectors.summingDouble(Employee::getSalary));
    System.out.println("sum = " + sum);
    System.out.println("------------------------");
    // 返回工资最大的用户
    Optional<Employee> optional = employees.stream()
        .collect(Collectors.maxBy((e1, e2) -> Double.compare(e1.getSalary(), e2.getSalary())));
    System.out.println("optional.get() = " + optional.get());
    System.out.println("------------------------");
    // 获取最小工资
    Optional<Double> minSalary = employees.stream()
        .map(Employee::getSalary)
        .collect(Collectors.minBy(Double::compare));
    System.out.println("minSalary.get() = " + minSalary.get());
}
```



分组：

```java
//分组
@Test
public void test6(){
    // 按状态分组
    Map<Employee.Status, List<Employee>> map = employees.stream()
        .collect(Collectors.groupingBy(Employee::getStatus));
    map.forEach((status, employees1) -> {
        System.out.println(status);
        System.out.println(employees1);
        System.out.println("------------------------");
    });
}
```

输出：

```
BUSY
[Employee{name='李四', age=38, salary=5555.99, status=BUSY}, Employee{name='田七', age=13, salary=778.3, status=BUSY}]
------------------------
FREE
[Employee{name='张三', age=18, salary=999.3, status=FREE}, Employee{name='赵六', age=55, salary=333.3, status=FREE}]
------------------------
VOCATION
[Employee{name='王五', age=17, salary=6636.66, status=VOCATION}]
------------------------
```



多级分组：

先按Status分，再按年龄段分

```java
// 多级分组
@Test
public void test7(){
    Map<Employee.Status, Map<String, List<Employee>>> map = employees.stream()
        .collect(Collectors.groupingBy(Employee::getStatus, Collectors.groupingBy((e) -> {
            if (e.getAge() <= 35) {
                return "青年";
            } else if (e.getAge() <= 50) {
                return "中年";
            } else {
                return "老年";
            }
        })));
    map.forEach((status, stringListMap) -> {
        System.out.println(status);
        stringListMap.forEach((s, employees1) -> {
            System.out.println(s);
            System.out.println(employees1);
        });
        System.out.println("--------------------------");
    });
}
```

输出：

```
VOCATION
青年
[Employee{name='王五', age=17, salary=6636.66, status=VOCATION}]
----------
BUSY
青年
[Employee{name='田七', age=13, salary=778.3, status=BUSY}]
----------
中年
[Employee{name='李四', age=38, salary=5555.99, status=BUSY}]
----------
FREE
青年
[Employee{name='张三', age=18, salary=999.3, status=FREE}]
----------
老年
[Employee{name='赵六', age=55, salary=333.3, status=FREE}]
----------
```



分区：

满足条件的一个区，不满足的一个区，也就是true和false

```java
// 分区
@Test
public void test8(){
    Map<Boolean, List<Employee>> map = employees.stream()
        .collect(Collectors.partitioningBy((e) -> e.getSalary() > 999));
    map.forEach((b, emp) -> {
        System.out.println(b);
        System.out.println(emp);
        System.out.println("-----------------");
    });
}
```

输出：

```
false
[Employee{name='赵六', age=55, salary=333.3, status=FREE}, Employee{name='田七', age=13, salary=778.3, status=BUSY}]
-----------------
true
[Employee{name='张三', age=18, salary=999.3, status=FREE}, Employee{name='李四', age=38, salary=5555.99, status=BUSY}, Employee{name='王五', age=17, salary=6636.66, status=VOCATION}]
-----------------
```



其他：

```java
@Test
public void test9(){
    DoubleSummaryStatistics dss = employees.stream()
            .collect(Collectors.summarizingDouble(Employee::getSalary));
    System.out.println(dss.getMax());
    System.out.println(dss.getMin());
    System.out.println(dss.getSum());
    System.out.println(dss.getAverage());
}
```

输出：

```java
6636.66
333.3
14303.55
2860.71
```



```java
@Test
public void test10(){
    String str = employees.stream()
        .map(Employee::getName)
        .collect(Collectors.joining());
    System.out.println("str = " + str);
    System.out.println("--------------------------");
    String str2 = employees.stream()
        .map(Employee::getName)
        .collect(Collectors.joining(","));
    System.out.println("str2 = " + str2);
    System.out.println("--------------------------");
    String str3 = employees.stream()
        .map(Employee::getName)
        .collect(Collectors.joining(",","<===","===>"));
    System.out.println("str3 = " + str3);
}
```

输出：

```
str = 张三李四王五赵六田七
--------------------------
str2 = 张三,李四,王五,赵六,田七
--------------------------
str3 = <===张三,李四,王五,赵六,田七===>
```

