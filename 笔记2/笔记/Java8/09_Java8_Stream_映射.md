---
2019-09-03 10:43:37

---

#

##映射

| 方法                            | 描述                                                         |
| ------------------------------- | ------------------------------------------------------------ |
| map(Function f)                 | 接收一个函数作为参数，该函数会被应用到每个元素上，并将其映射成一个新的元素。（将元素转换为其他形式或提取信息） |
| mapToDouble(ToDoubleFunction f) | 接收一个函数作为参数，该函数会被应用到每个元素上，产生一个新的DoubleStream |
| mapToInt(ToIntFunction f)       | 接收一个函数作为参数，该函数会被应用到每个元素上，产生一个新的IntStream |
| flatMap(Function f)             | 接收一个函数作为参数，将流中的每个值都换成另一个流，然后把所有流连接成一个流 |



- map

  ```java
  @Test
  public void test6(){
      List<String> list = Arrays.asList("aaa","bbb","ccc","ddd","eee");
      // 转大写
      list.stream()
          .map((x)->x.toUpperCase())
          .forEach(System.out::println);
      System.out.println("--------------------------");
      // 所有名字
      employees.stream()
          .map(Employee::getName)
          .forEach(System.out::println);
  }
  ```

- flatMap

  flat扁平的意思；

  map类似List中的add(Object obj)，而flatMap则类似addAll(Collection c)：

  map是把各个流当做一个元素整合成流，flatMap是把流中各个元素结果整合为一个流；

  {{a,a,a},{b,b,b}}与{a,a,a,b,b,b}

  ```java
  @Test
  public void test6(){
      List<String> list = Arrays.asList("aaa","bbb","ccc","ddd","eee");
      Stream<Stream<Character>> stream = list.stream()
          .map(TestStreamAPI2::filterCharacter);//{{a,a,a},{b,b,b}}
      stream.forEach((sm)->{
          sm.forEach(System.out::println);
      });
      System.out.println("---------------------------");
      Stream<Character> characterStream = list.stream()
          .flatMap(TestStreamAPI2::filterCharacter);//{a,a,a,b,b,b}
      characterStream.forEach(System.out::println);
  }
  public static Stream<Character> filterCharacter(String str){
      List<Character> list = new ArrayList<>();
      for (Character character : str.toCharArray()) {
          list.add(character);
      }
      return list.stream();
  }
  ```

  

