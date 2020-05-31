---
2019-09-02 15:11:32

---

#



Lambda是一个**匿名函数**，我们可以吧Lambda表达式理解为是**一段可以传递的代码**（将代码像数据一样进行传递）。可以写出更简洁、更灵活的代码。作为一种更紧凑的代码风格，使得Java的语言表达能力得到了提升。



##

匿名内部类对比

JDK8以前：

```java
/**
 * 原来的匿名内部类
 */
@Test
public void test1(){
    Comparator<Integer> com = new Comparator<Integer>() {
        public int compare(Integer o1, Integer o2) {
            return Integer.compare(o1,o2);
        }
    };
    TreeSet<Integer> ts = new TreeSet<>(com);
}
```

JKD8以后：

```java
/**
 * Lambda表达式
 */
@Test
public void test2(){
    Comparator<Integer> com = (x,y) -> Integer.compare(x,y);
    TreeSet<Integer> ts = new TreeSet<>(com);
}
```



##List对比

两个需求：

1. 获取当前公司中员工年龄大于35的员工信息
2. 获取当前公司中工资大于5000的员工信息

###准备工作

准备List变量：

```java
List<Employee> employees = Arrays.asList(
    new Employee("张三",18,999.3),
    new Employee("李四",38,5555.99),
    new Employee("王五",17,6636.66),
    new Employee("赵六",55,333.3),
    new Employee("田七",13,778.3)
);
```

准备Employee类：

```java
public class Employee {

    private String name;
    private Integer age;
    private Double salary;

    public Employee() {
    }

    @Override
    public String toString() {
        return "Employee{" +
                "name='" + name + '\'' +
                ", age=" + age +
                ", salary=" + salary +
                '}';
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

    public Employee(String name, Integer age, Double salary) {
        this.name = name;
        this.age = age;
        this.salary = salary;
    }
}
```



###普通方式

```java
/**
 * 需求：获取当前公司中员工年龄大于35的员工信息
 */
public List<Employee> filterEmployees(List<Employee> list){
    List<Employee> emps = new ArrayList<>();
    for (Employee emp : list) {
        if(emp.getAge()>35){
            emps.add(emp);
        }
    }
    return emps;
}
@Test
public void test3(){
    List<Employee> emps = filterEmployees(this.employees);
    for (Employee emp : emps) {
        System.out.println(emp);
    }
}
/**
 * 需求：获取当前公司中工资大于5000的员工信息
 */
public List<Employee> filterEmployee2(List<Employee> list){
    List<Employee> emps = new ArrayList<>();
    for (Employee emp : list) {
        if(emp.getSalary()>=5000){
            emps.add(emp);
        }
    }
    return emps;
}
```



###优化1（策略设计模式）

可以看到随着需求不断增多，代码重复率很高，可以用一些设计模式来解决（这里用**策略设计模式**）：

定义一个接口MyPredicate：

```java
public interface MyPredicate<T> {

    public boolean test(T t);

}
```

实现接口：

```java
public class FilterEmployeeByAge implements MyPredicate<Employee>{

    @Override
    public boolean test(Employee employee) {
        return employee.getAge()>=35;
    }
}
```

测试：

```java
/**
 * 优化方式1：策略设计模式
 */
@Test
public void test4(){
    List<Employee> employees = filterEmployee(this.employees, new FilterEmployeeByAge());
    for (Employee emp : employees) {
        System.out.println(emp);
    }
}

public List<Employee> filterEmployee(List<Employee> list,MyPredicate<Employee> mp){
    List<Employee> employees = new ArrayList<>();
    for (Employee emp : list) {
        if(mp.test(emp)){
            employees.add(emp);
        }
    }
    return employees;
}
```

如果有其他需求，只要再定义一个新的类实现MyPredicate，然后直接传入这个对象即可：

FilterEmployeeByAge类：

```java
public class FilterEmployeeByAge implements MyPredicate<Employee>{

    @Override
    public boolean test(Employee employee) {
        return employee.getAge()>=35;
    }
}
```

测试：

```java
/**
 * 优化方式1：策略设计模式
 */
@Test
public void test4(){
    List<Employee> employees = filterEmployee(this.employees, new FilterEmployeeByAge());
    for (Employee emp : employees) {
        System.out.println(emp);
    }
    System.out.println("--------------------------------");
    List<Employee> employees1 = filterEmployee(this.employees, new FilterEmployeeBySalary());
    for (Employee emp : employees1) {
        System.out.println(emp);
    }
}
```



###优化2（匿名内部类）

还是用策略设计模式，只不过不需要定义更多的类实现逻辑，只需要用匿名内部类来实现：

```java
// 优化方式2：匿名内部类
@Test
public void test5(){
    List<Employee> emps1 = filterEmployee(this.employees, new MyPredicate<Employee>() {
        @Override
        public boolean test(Employee employee) {
            return employee.getAge() >= 35;
        }
    });
    for (Employee emp : emps1) {
        System.out.println(emp);
    }
    System.out.println("------------------------");
    List<Employee> emps2 = filterEmployee(this.employees, new MyPredicate<Employee>() {
        @Override
        public boolean test(Employee employee) {
            return employee.getSalary() > 5000;
        }
    });
    for (Employee emp : emps2) {
        System.out.println(emp);
    }
}
```



###优化3（Lambda表达式）

基于上面的优化2改造：

```java
// 优化方式3：Lambda表达式
@Test
public void test6(){
    List<Employee> emps1 = filterEmployee(this.employees, (e) -> e.getAge() >= 35);
    emps1.forEach(System.out::println);
    System.out.println("-----------------");
    List<Employee> emps2 = filterEmployee(this.employees, (e) -> e.getSalary() > 5000);
    emps2.forEach(System.out::println);
}
```

可以看到一个需求只需要两行就可以实现；



###优化4（StreamAPI）

与上面的几种方式都无关，包括接口也不需要：

```java
// 优化方式4：Stream API
@Test
public void test7(){
    employees.stream()
        .filter((e)->e.getAge()>=35)
        .forEach(System.out::println);
}
```

可以看到一个需求实际上只用了一行代码；



还可以进行数据条数限制：

```java
// 优化方式4：Stream API
@Test
public void test7(){
    employees.stream()
        .filter((e)->e.getAge()>=35)
        .limit(1)
        .forEach(System.out::println);
}
```

这样就只取出一条；



增加需求，提取出所有的名字：

```java
// 优化方式4：Stream API
@Test
public void test7(){
    employees.stream()
        .map(Employee::getName)
        .forEach(System.out::println);
}
```

