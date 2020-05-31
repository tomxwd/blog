---
2019-09-04 15:06:24

---

#

> [jdk8中文文档](http://www.matools.com/api/java8)

传统的时间格式化都存在线程安全问题；

新的时间格式化类在java.time及其子包下；

- java.time：
  - 一些本地日期（Local开头的）
  - 时间戳Instant
  - Duration两个时间之间的间隔
  - Period两个日期之间的间隔
  - OffsetTime时间偏移量
- java.time.chrono
  - 日期格式的，如日本、台湾的等；
- java.time.format
  - 日期格式化的
- java.time.temporal
  - 时间运算的
- java.time.zone
  - 时区运算的

##传统多线程问题

```java
@Test
public void test(){
    SimpleDateFormat sdf = new SimpleDateFormat("yyyyMMdd");
    Callable<Date> call = new Callable<Date>() {
        @Override
        public Date call() throws Exception {
            return sdf.parse("20161218");
        }
    };
    ExecutorService pool = Executors.newFixedThreadPool(10);
    List<Future<Date>> results = new ArrayList<>();
    for (int i = 0; i < 10; i++) {
        results.add(pool.submit(call));
    }
    results.stream()
            .forEach(dateFuture -> {
                try {
                    System.out.println("dateFuture.get() = " + dateFuture.get());
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (ExecutionException e) {
                    e.printStackTrace();
                }
            });
    pool.shutdown();
}
```

##使用ThreadLocal解决多线程安全问题

需要创建一个DateFormatThreadLocal类：

```java
public class DateFormatThreadLocal {

    private static final ThreadLocal<DateFormat> df = new ThreadLocal<DateFormat>(){
        @Override
        protected DateFormat initialValue() {
            return new SimpleDateFormat("yyyyMMdd");
        }
    };

    public static Date convert(String source) throws ParseException {
        return df.get().parse(source);
    }

}
```

测试：

```java
@Test
public void test2(){
    Callable<Date> call = new Callable<Date>() {
        @Override
        public Date call() throws Exception {
            return DateFormatThreadLocal.convert("20161218");
        }
    };
    ExecutorService pool = Executors.newFixedThreadPool(10);
    List<Future<Date>> results = new ArrayList<>();
    for (int i = 0; i < 10; i++) {
        results.add(pool.submit(call));
    }
    results.stream()
            .forEach(dateFuture -> {
                try {
                    System.out.println("dateFuture.get() = " + dateFuture.get());
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (ExecutionException e) {
                    e.printStackTrace();
                }
            });
    pool.shutdown();
}
```

##新的日期类解决线程安全问题

```java
@Test
public void test3() throws ExecutionException, InterruptedException {

    // DateTimeFormatter dft = DateTimeFormatter.ISO_DATE_TIME;
    DateTimeFormatter dft = DateTimeFormatter.ofPattern("yyyyMMdd");

    Callable<LocalDate> call = new Callable<LocalDate>() {
        @Override
        public LocalDate call() throws Exception {
            return LocalDate.parse("20161218",dft);
        }
    };
    ExecutorService pool = Executors.newFixedThreadPool(10);
    List<LocalDate> list = new ArrayList<>();
    for (int i = 0; i < 10; i++) {
        list.add(pool.submit(call).get());
    }
    list.stream()
        .forEach(System.out::println);
}
```