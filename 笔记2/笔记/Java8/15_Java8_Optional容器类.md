---
2019-09-04 10:16:06

---

#

##Optional类

Optional\<T>类（java.util.Optional）是一个容器类，代表一个值存在或不存在，原来用null表示一个值不存在，现在Optional可以更好的表达这个概念。并且可以避免空指针异常。

**常用方法：**

- Optional.of(T t)：创建一个Optional实例

  ```java
  @Test
  public void test1(){
      Optional<Employee> op = Optional.of(new Employee());
      // 虽然说可以传入null，但是如果这样做，在这里就会抛出异常，可以做到快速定位
      Optional<Employee> op2 = Optional.of(null);
      Employee emp = op.get();
      System.out.println("emp = " + emp);
  }
  ```

- Optional.empty()：创建一个Optional实例

  ```java
  @Test
  public void test2(){
      Optional<Employee> op = Optional.empty();
      System.out.println(op.get());
  }
  ```

- Optional.ofNullable(T t)：若t不为null，创建Optional实例，否则创建空实例

  源码中是`return value == null ? empty() : of(value);`

  value为null就调用empty()；

  value不为null就调用of(value)；

  ```java
  @Test
  public void test3(){
      Optional<Employee> op = Optional.ofNullable(new Employee());
      System.out.println(op.get());
      Optional<Employee> op2 = Optional.ofNullable(null);
      System.out.println(op2.get());
  }
  ```

- isPresent()：判断是否包含值

  ```java
  @Test
  public void test4(){
      Optional<Employee> op = Optional.ofNullable(new Employee());
      if(op.isPresent()){
          System.out.println(op.get());
      }else{
          System.out.println("无值");
      }
      Optional<Employee> op2 = Optional.ofNullable(null);
      if(op2.isPresent()){
          System.out.println(op.get());
      }else{
          System.out.println("无值");
      }
  }
  ```

- orElse(T t)：如果调用对象包含值，返回该值，否则返回t

  ```java
  @Test
  public void test5(){
      Optional<Employee> op = Optional.ofNullable(null);
      Employee employee = op.orElse(new Employee());
      System.out.println("employee = " + employee);
  }
  ```

- orElseGet(Supplier s)：如果调用对象包含值，返回该值，否则返回s获取的值

  ```java
  @Test
  public void test6() {
      Optional<Employee> op = Optional.ofNullable(null);
      Employee employee = op.orElseGet(() -> new Employee());
      System.out.println("employee = " + employee);
  }
  ```

- map(Function f)：如果有值对其处理，并返回处理后的Optional，否则返回Optional.empty()

  ```java
  @Test
  public void test7(){
      Optional<Employee> op = Optional.ofNullable(new Employee("张三",12,12.2, Employee.Status.BUSY));
      Optional<String> str = op.map(employee -> employee.getName());
      System.out.println("str.get() = " + str.get());
  }
  ```

- flatMap(Function mapper)：与map类似，要求返回值必须是Optional

  进一步防止空指针异常

  ```java
  @Test
  public void test8(){
      Optional<Employee> op = Optional.ofNullable(new Employee("张三",12,12.2, Employee.Status.BUSY));
      Optional<String> str = op.flatMap(employee -> Optional.of(employee.getName()));
      System.out.println("str.get() = " + str.get());
  }
  ```



综合测试：

创建Man类：

```java
public class Man {

    private Godness godness;

    @Override
    public String toString() {
        return "Man{" +
                "godness=" + godness +
                '}';
    }

    public Godness getGodness() {
        return godness;
    }

    public void setGodness(Godness godness) {
        this.godness = godness;
    }

    public Man(Godness godness) {
        this.godness = godness;
    }

    public Man() {
    }
}
```

创建Godness类：

```java
public class Godness {

    @Override
    public String toString() {
        return "Godness{" +
                "name='" + name + '\'' +
                '}';
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Godness() {
    }

    public Godness(String name) {
        this.name = name;
    }

    private String name;

}
```

测试1（传统模式）：

```java
// 需求获取man中godness的名字
public String getGodnessName(Man man){
    if(man!=null){
        Godness gn = man.getGodness();
        if(gn!=null){
            return gn.getName();
        }
    }
    return "默认名";
}

@Test
public void test9(){
    Man man = new Man();
    String godnessName = getGodnessName(man);
    System.out.println("godnessName = " + godnessName);
}
```

测试2（Optional）：

定义一个NewMan类，用Optional来装Godness；

```java
public class NewMan {

    private Optional<Godness> godness = Optional.empty();

    @Override
    public String toString() {
        return "NewMan{" +
                "godness=" + godness +
                '}';
    }

    public Optional<Godness> getGodness() {
        return godness;
    }

    public void setGodness(Optional<Godness> godness) {
        this.godness = godness;
    }

    public NewMan() {
    }

    public NewMan(Optional<Godness> godness) {
        this.godness = godness;
    }
}
```

测试：

```java
// 需求获取NewMan中godness的名字
public String getGodnessName2(Optional<NewMan> man){
    return man.orElse(new NewMan())
        .getGodness()
        .orElse(new Godness("默认"))
        .getName();
}

@Test
public void test10(){
    String godnessName = getGodnessName2(Optional.ofNullable(null));
    System.out.println("godnessName = " + godnessName);
}
```



