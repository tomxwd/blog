---
2019-09-02 17:34:41

---

#



## 四大内置函数式接口

| 函数式接口                      | 参数类型 | 返回类型 | 用途                                                         |
| ------------------------------- | -------- | -------- | ------------------------------------------------------------ |
| Consumer\<T\><br />消费型接口   | T        | void     | 对类型为T的对象应用操作。<br />包含方法：void accept(T t)；  |
| Supplier\<T\><br />供给型接口   | 无       | T        | 返回类型为T的对象。<br />包含方法：T get()；                 |
| Function\<T,R\><br />函数型接口 | T        | R        | 对类型为T的对象应用操作，并返回结果。<br />结果是R类型的对象。<br />包含方法：R apply(T t)； |
| Predicate\<T\><br />断定型接口  | T        | boolean  | 确定类型为T的对象是否满足某约束，并返回boolean值。<br />包含方法：boolean test(T t)； |



### 内置函数式接口测试

1. Consumer\<T\>：消费型接口

   - void accept(T t)；

   ```java
   // Comsumer<T>
   @Test
   public void test1(){
       happy(10000,(m) -> System.out.println("消费" + m + "元"));
   }
   
   public void happy(double money, Consumer<Double> consumer){
       consumer.accept(money);
   }
   ```

2. Supplier\<T\>：供给型接口

   - T get()；

   ```java
   // Supplier<T>
   @Test
   public void test2(){
       getNumList(10,() -> (int)(Math.random()*100)).stream().forEach(System.out::println);
   }
   
   // 产生指定整数，并放入集合中
   public List<Integer> getNumList(int num, Supplier<Integer> supplier){
       List<Integer> list = new ArrayList<>();
       for (int i = 0; i < num; i++) {
           list.add(supplier.get());
       }
       return list;
   }
   ```

3. Function<T,R>：函数型接口

   - R apply(T t)；

   ```java
   // Function<T,R>
   @Test
   public void test3(){
       System.out.println(strHandler("\t\t\thello\t\t", (x) -> x.trim()));
       System.out.println(strHandler("\t\t\thello\t\t", (x) -> x.toUpperCase()));
       System.out.println(strHandler("\t\t\thello\t\t", (x) -> x.substring(2,4)));
   }
   
   // 需求：用于处理字符串
   public String strHandler(String str, Function<String,String> fun){
       return fun.apply(str);
   }
   ```

4. Predicate\<T\>：断言型接口

   - boolean test(T t)；
   
   ```java
   // Predicate<T>
   @Test
   public void test4(){
       List<String> list = Arrays.asList("字符1","字符2","字符312");
       // 过滤出长度大于3的字符
       List<String> strings = filterStr(list,(x)->x.length()>3);
       strings.stream().forEach(System.out::println);
   }
   
   // 需求： 将满足条件的字符串，放入到集合中去
   public List<String> filterStr(List<String> list, Predicate<String> predicate){
       List<String> strList = new ArrayList<>();
       list.stream().forEach(e -> {
           if (predicate.test(e)) {
               strList.add(e);
           }
       });
       return strList;
   }
   ```



##其他接口

| 函数式接口                                                   | 参数类型                  | 返回类型                  | 用途                                                         |
| ------------------------------------------------------------ | ------------------------- | ------------------------- | ------------------------------------------------------------ |
| BiFunction<T,U,R>                                            | T,U                       | R                         | 对类型为T，U参数应用操作，返回R类型的结果，包含方法为R apply(T t,U u)； |
| UnaryOperator\<T\><br />（Function子接口）                   | T                         | T                         | 对类型为T的对象进行一元运算，并返回T类型的结果。包含方法为T apply(T t)； |
| BinaryOperator\<T><br />（BiFunction子接口）                 | T,T                       | T                         | 对类型为T的对象进行二元运算，并返回T类型的结果。包含方法为T apply(T t1,T t2)； |
| BiConsumer<T,U>                                              | T,U                       | void                      | 对类型为T，U参数应用操作。包含方法为T apply(T t,U u)；       |
| ToIntFunction\<T><br />ToLongFunction\<T><br />ToDoubleFunction\<T> | T                         | int<br />long<br />double | 分别计算int、long、double值的函数；                          |
| IntFunction\<R><br />LongFunction\<R><br />DoubleFunction\<R> | int<br />long<br />double | R                         | 参数分别为int、long、double类型的函数                        |