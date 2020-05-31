---
2019-09-03 08:31:53

---

#

##方法引用

方法引用：若Lambda体中的内容有方法已经实现了，我们可以使用方法引用（可以理解为方法引用是Lambda表达式的另外一种表现形式）

###主要的三种语法格式

1. 对象::实例方法名

   ```java
   // 对象::实例方法名
   @Test
   public void test1(){
       Consumer<String> con = (x) -> System.out.println(x);
       con.accept("123");
       Consumer<String> con1 = System.out::println;
       con1.accept("1231");
   }
   
   @Test
   public void test2(){
       Employee employee = new Employee();
       Supplier<String> sup = () -> employee.getName();
       Supplier<Integer> sup1 = employee::getAge;
       Integer integer = sup1.get();
       System.out.println("integer = " + integer);
   }
   ```

2. 类::静态方法名

   ```java
   // 类::静态方法名
   @Test
   public void test3(){
       Comparator<Integer> com = (x,y) -> Integer.compare(x,y);
       Comparator<Integer> com1 = Integer::compare;
   }
   ```

3. 类::实例方法名

   ```java
   // 类::实例方法名
   @Test
   public void test4(){
       BiPredicate<String ,String> bp = (x,y) -> x.equals(y);
       BiPredicate<String ,String> bp1 = String::equals;
   }
   ```



###注意：

1. Lambda体中调用方法的参数列表与返回值类型，要与函数式接口中抽象方法的参数列表和返回值类型保持一致。
2. 若Lambda参数列表中第一个参数是实例方法的调用者，而第二个参数是实例方法的参数时，才可以使用ClassName::method（类::实例方法名）



##构造器引用

格式：

ClassName::new

Supplier参数列表为空，返回值为T，所以是调用Employee的无参构造器：

```java
// 构造器引用
@Test
public void test5(){
    Supplier<Employee> sup = () -> new Employee();
    // 构造器引用方式
    Supplier<Employee> sup1 = Employee::new;
}
```

新增Employee参数列表为name的构造器：

```java
public Employee(String name) {
    this.name = name;
}
```

Function参数列表类型为T，返回值类型为R，所以调用Employee的一个参数的构造器：

```java
@Test
public void test6(){
    Function<String,Employee> fun1 = (x) -> new Employee(x);
    Function<String,Employee> fun2 = Employee::new;
}
```

如果想用BiFunction<T,U,R>这种两个参数一个返回值的，必须要先去定义构造方法才可以使用，否则编译期间就报错；

```java
BiFunction<String,String,Employee> fun3 = Employee::new;
```



###注意

1. 需要调用的构造器的参数列表要与函数式接口中抽象方法的参数列表保持一致；



##数组引用

Type[]::new

```java
// 数组引用
@Test
public void test7(){
    Function<Integer,String[]> fun = (x) -> new String[x];
    String[] strs = fun.apply(4);
    System.out.println(strs.length);

    Function<Integer,String[]> fun2 = String[]::new;
    String[] str2 = fun2.apply(6);
    System.out.println(str2.length);
}
```

