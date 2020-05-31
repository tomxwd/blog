---
2019-09-03 16:33:41

---

#

1. 给定一个数字列表，如何返回一个由每个数的平方构成的列表呢？

   给定【1,2,3,4,5】，应该返回【1,4,9,16,25】

   ```java
   @Test
   public void test1(){
       Integer[] nums = {1, 2, 3, 4, 5};
       // map
       Arrays.stream(nums)
           .map((x)->x*x).forEach(System.out::println);
       // reduce
       Integer reduce = Arrays.stream(nums)
           .reduce(1, (integer, integer2) -> {
               System.out.println(integer2*integer2);
               return integer2*integer2;
           });
   }
   ```

2. 怎么用map和reduce方法数一数流中有多少个Employee呢？

   ```java
   List<Employee> employees = Arrays.asList(
       new Employee("张三",18,999.3, Employee.Status.FREE),
       new Employee("李四",38,5555.99,Employee.Status.BUSY),
       new Employee("王五",17,6636.66, Employee.Status.VOCATION),
       new Employee("赵六",55,333.3, Employee.Status.FREE),
       new Employee("田七",13,778.3, Employee.Status.BUSY)
   );
   @Test
   public void test2(){
       Optional<Integer> reduce = employees.stream()
               .map((e) -> 1)
               .reduce(Integer::sum);
       System.out.println("reduce.get() = " + reduce.get());
   }
   ```



准备：

交易员Trader类：

```java
public class Trader {

    private String name;
    private String city;

    public Trader() {
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;

        Trader trader = (Trader) o;

        if (name != null ? !name.equals(trader.name) : trader.name != null) return false;
        return city != null ? city.equals(trader.city) : trader.city == null;
    }

    @Override
    public int hashCode() {
        int result = name != null ? name.hashCode() : 0;
        result = 31 * result + (city != null ? city.hashCode() : 0);
        return result;
    }

    @Override
    public String toString() {
        return "Trader{" +
                "name='" + name + '\'' +
                ", city='" + city + '\'' +
                '}';
    }

    public String getCity() {
        return city;
    }

    public void setCity(String city) {
        this.city = city;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Trader(String name, String city) {
        this.name = name;
        this.city = city;
    }
}
```

交易类Transaction：

```java
public class Transaction {

    private Trader trader;
    private int year;
    private int value;

    @Override
    public String toString() {
        return "Transaction{" +
                "trader=" + trader +
                ", year=" + year +
                ", value=" + value +
                '}';
    }

    public Trader getTrader() {
        return trader;
    }

    public void setTrader(Trader trader) {
        this.trader = trader;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;

        Transaction that = (Transaction) o;

        if (year != that.year) return false;
        if (value != that.value) return false;
        return trader != null ? trader.equals(that.trader) : that.trader == null;
    }

    @Override
    public int hashCode() {
        int result = trader != null ? trader.hashCode() : 0;
        result = 31 * result + year;
        result = 31 * result + value;
        return result;
    }

    public int getYear() {
        return year;
    }

    public void setYear(int year) {
        this.year = year;
    }

    public int getValue() {
        return value;
    }

    public void setValue(int value) {
        this.value = value;
    }

    public Transaction(Trader trader, int year, int value) {
        this.trader = trader;
        this.year = year;
        this.value = value;
    }

    public Transaction() {
    }
}
```

准备好的测试类TestTransaction：

```java
public class TestTransaction {

    List<Transaction> transactions = null;

    @Before
    public void before(){
        Trader raoul = new Trader("Raoul","Cambridge");
        Trader mario = new Trader("Mario","Milan");
        Trader alan = new Trader("Alan","Cambridge");
        Trader brian = new Trader("Brain","Cambridge");
        transactions = Arrays.asList(
                new Transaction(brian,2011,300),
                new Transaction(raoul,2012,1000),
                new Transaction(raoul,2011,400),
                new Transaction(mario,2012,710),
                new Transaction(mario,2012,700),
                new Transaction(alan, 2012,950)
        );
    }

}
```

1. 找出2011年发生的所有交易，并按交易额排序（从低到高）

   ```java
   //1. 找出2011年发生的所有交易，并按交易额排序（从低到高）
   @Test
   public void test1(){
       transactions.stream()
           .filter(transaction -> transaction.getYear()==2011)
           .sorted((o1, o2) -> Integer.compare(o1.getValue(),o2.getValue()))
           .forEach(System.out::println);
   }
   ```

2. 交易员都在哪些不同的城市工作过？

   ```java
   //2. 交易员都在哪些不同的城市工作过？
   @Test
   public void test2(){
       transactions.stream()
           .map(Transaction::getTrader)
           .map(Trader::getCity)
           .distinct()
           .forEach(System.out::println);
   }
   ```

3. 查找所有来自剑桥的交易员，并按姓名排序

   ```java
   //3. 查找所有来自剑桥的交易员，并按姓名排序
   @Test
   public void test3(){
       transactions.stream()
           .map(Transaction::getTrader)
           .distinct()
           .filter(trader -> trader.getCity().equals("Cambridge"))
           .sorted((o1, o2) -> o1.getName().compareTo(o2.getName()))
           .forEach(System.out::println);
   }
   ```

4. 返回所有交易员的姓名字符串，按字母顺序排序

   ```java
   //4. 返回所有交易员的姓名字符串，按字母顺序排序
   @Test
   public void test4(){
       transactions.stream()
           .map(transaction -> transaction.getTrader().getName())
           .distinct()
           .sorted((o1, o2) -> o1.compareTo(o2))
           .forEach(System.out::println);
       System.out.println("------------------");
       String str = transactions.stream()
           .map(transaction -> transaction.getTrader().getName())
           .distinct()
           .sorted((o1, o2) -> o1.compareTo(o2))
           .reduce("", String::concat);
       System.out.println("str = " + str);
       System.out.println("------------------");
       transactions.stream()
           .map(transaction -> transaction.getTrader().getName())
           .distinct()
           .flatMap(TestTransaction::filterCharacter)
           .sorted((o1, o2) -> o1.compareToIgnoreCase(o2))
           .forEach(System.out::print);
   }
   
   public static Stream<String> filterCharacter(String str){
       List<String> list = new ArrayList<>();
       for (Character c : str.toCharArray()) {
           list.add(c.toString());
       }
       return list.stream();
   }
   ```

5. 有没有交易员是在米兰工作的？

   ```java
   //5. 有没有交易员是在米兰工作的？
   @Test
   public void test5(){
       boolean b = transactions.stream()
           .anyMatch(transaction -> transaction.getTrader().getCity().equals("Milan"));
       System.out.println("b = " + b);
   }
   ```

6. 打印生活在剑桥的交易员的所有交易额

   ```java
   //6. 打印生活在剑桥的交易员的所有交易额
   @Test
   public void test6(){
       Optional<Integer> values = transactions.stream()
           .filter(transaction -> transaction.getTrader().getCity().equals("Cambridge"))
           .map(Transaction::getValue)
           .reduce(Integer::sum);
       System.out.println("values.get() = " + values.get());
   }
   ```

7. 所有交易中，最高的交易额是多少？

   ```java
   //7. 所有交易中，最高的交易额是多少？
   @Test
   public void test7(){
       Optional<Integer> max = transactions.stream()
           .map(transaction -> transaction.getValue())
           .max((o1, o2) -> Integer.compare(o1, o2));
       System.out.println("max.get() = " + max.get());
   }
   ```

8. 找到交易额最小的交易

   ```java
   //8. 找到交易额最小的交易
   @Test
   public void test8(){
       Optional<Transaction> min = transactions.stream()
           .min((o1, o2) -> Integer.compare(o1.getValue(), o2.getValue()));
       System.out.println("min.get() = " + min.get());
   }
   ```