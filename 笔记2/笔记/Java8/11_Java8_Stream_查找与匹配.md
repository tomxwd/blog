---
2019-09-03 13:57:25

---

#

##Stream的终止操作

终端操作会从流的流水线生成结果。

其结果可以是任何不是流的值，例如：List、Integer，甚至是void。

###准备工作

改造的Employee类：

```java
public class Employee {

    private String name;
    private Integer age;
    private Double salary;
    private Status status;

    public Employee() {
    }

    public Employee(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public Double getSalary() {
        return salary;
    }

    public void setSalary(Double salary) {
        this.salary = salary;
    }

    public Employee(String name, Integer age, Double salary, Status status) {
        this.name = name;
        this.age = age;
        this.salary = salary;
        this.status = status;
    }

    @Override
    public String toString() {
        return "Employee{" +
                "name='" + name + '\'' +
                ", age=" + age +
                ", salary=" + salary +
                ", status=" + status +
                '}';
    }

    public Status getStatus() {
        return status;
    }

    public void setStatus(Status status) {
        this.status = status;
    }

    public Employee(String name, Integer age, Double salary) {
        this.name = name;
        this.age = age;
        this.salary = salary;
    }
    public enum Status{
        FREE,
        BUSY,
        VOCATION;
    }
}
```

准备的数据变量：

```java
List<Employee> employees = Arrays.asList(
    new Employee("张三",18,999.3, Employee.Status.FREE),
    new Employee("李四",38,5555.99,Employee.Status.BUSY),
    new Employee("王五",17,6636.66, Employee.Status.VOCATION),
    new Employee("赵六",55,333.3, Employee.Status.FREE),
    new Employee("田七",13,778.3, Employee.Status.BUSY)
);
```



###查找与匹配

| 方法                   | 描述                     |
| ---------------------- | ------------------------ |
| allMatch(Predicate p)  | 检查是否匹配所有元素     |
| anyMatch(Predicate p)  | 检查是否至少匹配一个元素 |
| noneMatch(Predicate p) | 检查是否没有匹配所有元素 |
| findFirst()            | 返回第一个元素           |
| findAny()              | 返回当前流中的任意元素   |

```java
@Test
public void test1(){
    // allMatch
    boolean b1 = employees.stream()
        .allMatch((e) -> e.getStatus().equals(Employee.Status.BUSY));
    System.out.println("b1 = " + b1);
    // anyMatch
    boolean b2 = employees.stream()
        .anyMatch((e) -> e.getStatus().equals(Employee.Status.BUSY));
    System.out.println("b2 = " + b2);
    // noneMatch
    boolean b3 = employees.stream()
        .noneMatch((e) -> e.getStatus().equals(Employee.Status.BUSY));
    System.out.println("b3 = " + b3);
    // findFirst
    Optional<Employee> op = employees.stream()
        .sorted((e1, e2) -> Double.compare(e1.getSalary(), e2.getSalary()))
        .findFirst();
    System.out.println("op.get() = " + op.get());
    // findAny
    Optional<Employee> any = employees.parallelStream()
        .filter(e -> e.getStatus().equals(Employee.Status.FREE))
        .findAny();
    System.out.println("any.get() = " + any.get());
}
```

注意：

1. findFirst()方法和findAny()方法会返回一个Optional容器对象，可以用其中的get()方法获取结果，也可以用orElse(T)来指定为null的时候返回什么对象；
2. findAny()方法，如果是用stream()来创建流对象的话，是串行的，也就是说会一直得到第一个符合的元素，这里需要注意要用并行的，也就是parallelStream()来创建流对象；

| 方法                  | 描述                                                         |
| --------------------- | ------------------------------------------------------------ |
| count()               | 返回流中元素的个数                                           |
| max(Comparator c)     | 返回流中最大值                                               |
| min(Comparator c)     | 返回流中最小值                                               |
| forEach(Comparator c) | **内部迭代**（使用Collection接口需要用户去做迭代，称为外部迭代。相反，Stream API使用内部迭代--它帮你把迭代做了） |

```java
@Test
public void test2(){
    // count
    long count = employees.stream()
            .count();
    System.out.println("count = " + count);
    // max 薪资最高的Employee
    Optional<Employee> max = employees.stream()
            .max((o1, o2) -> o1.getSalary().compareTo(o2.getSalary()));
    System.out.println("max.get() = " + max.get());
    // min 年龄最小的Employee
    Optional<Employee> min = employees.stream()
            .min((o1, o2) -> o1.getAge().compareTo(o2.getAge()));
    System.out.println("min.get() = " + min.get());
    // 获取工资最少是多少
    Optional<Double> minSalary = employees.stream()
            .map(Employee::getSalary)
            .min(Double::compare);
    System.out.println("minSalary.get() = " + minSalary.get());
}
```