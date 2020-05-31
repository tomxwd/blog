---
2019-09-02 17:09:18

---

#

准备工作：

Employee类：

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

测试类：

```java
public class TestLambda {

    List<Employee> employees = Arrays.asList(
            new Employee("张三",18,999.3),
            new Employee("李四",38,5555.99),
            new Employee("王五",17,6636.66),
            new Employee("赵六",55,333.3),
            new Employee("田七",13,778.3)
    );
    
}
```

1. 调用Collections.sort()方法，通过定制排序比较两个Employee（先按年龄比，年龄相同的按姓名比），使用Lambda作为参数传递。

   ```java
   @Test
   public void test1(){
       Collections.sort(employees,(e1,e2) -> {
           if(e1.getAge()==e2.getAge()){
               return e1.getName().compareTo(e2.getName());
           }else{
               return Integer.compare(e1.getAge(),e2.getAge());
           }
       });
       employees.forEach(System.out::println);
   }
   ```

2. 

   - 声明函数式接口，接口中声明抽象方法，public String getValue(String str)；

     ```java
     @FunctionalInterface
     public interface MyFunction {
     
         String getValue(String str);
     
     }
     ```

   - 声明类TestLambda，类中编写方法使用接口作为参数；

     方法：

     ```java
     // 需求：用于处理字符串
     public String strHandler(String str,MyFunction mf){
         return mf.getValue(str);
     }
     ```

   - 将一个字符串转换成大写，并作为方法的返回值；

   - 将一个字符串去除空格，并作为方法的返回值；

   - 再将一个字符串的第2个和第4个索引位置进行截取子串；

     ```java
     @Test
     public void test2(){
         // 小写转大写
         String s = strHandler("xxxxxwwww", (x) -> x.toUpperCase());
         System.out.println("s = " + s);
         // 去除空格
         s = strHandler("\t\t\txxxxxwwww",(x) -> x.trim());
         System.out.println("s = " + s);
         // 2-4索引位置截取子串
         s = strHandler("abcdefg",(x) -> x.substring(2,4));
         System.out.println("s = " + s);
     }
     ```

3. 

   - 声明一个带两个泛型的函数式接口，泛型类型为`<T,R>`T为参数，R为返回值；

   - 接口中声明对应抽象方法；

     ```java
     @FunctionalInterface
     public interface MyFunction2<T,R> {
     
         R getValue(T t1,T t2);
     
     }
     ```

   - 在TestLambda类中声明方法，使用接口作为参数，计算两个long类型参数的和；

   - 再计算两个long类型参数的乘积；

     ```java
     // 需求：对两个Long类型进行处理
     public void op(Long l1,Long l2,MyFunction2<Long,Long> mf){
         System.out.println(mf.getValue(l1,l2));
     }
     
     @Test
     public void test3(){
         op(100L, 200L, (x, y) -> x + y);
         op(100L, 200L, (x, y) -> x * y);
     }
     ```

     