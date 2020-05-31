---
2019-09-04 16:15:21

---

#

##使用LocalDate、LocalTime、LocalDateTime

LocalDate、LocalTime、LocalDateTime类的实例是**不可变的对象**，分别表示使用ISO-8601日历系统的日期、时间、日期和时间。

他们提供了简单的日期或时间，并不包含当前的时间信息。也不包含与时区相关的信息。

注意：**ISO-8601日历系统是国际标准化组织制定的现代公民的日期和时间的表示法**；

测试：

```java
public class TestLocalDateTime {
    
    // 1. LocalDate  LocalTime  LocalDateTime
    
    /**
     * now()方法
     */
    @Test
    public void test1(){
        LocalDate ld = LocalDate.now();
        System.out.println(ld);
        LocalTime lt = LocalTime.now();
        System.out.println(lt);
        LocalDateTime ldt = LocalDateTime.now();
        System.out.println(ldt);
    }

    /**
     * of方法
     */
    @Test
    public void test2(){
        LocalDateTime ldt = LocalDateTime.of(2019, 12, 30, 2, 2, 3,1234);
        System.out.println("ldt = " + ldt);
    }

    /**
     * plus方法
     * minus方法
     * plusXxx加对应的Xxx
     * minusXxx减对应的Xxx
     */
    @Test
    public void test3(){
        LocalDateTime ldt = LocalDateTime.of(2019, 12, 30, 2, 2, 3,1234);
        LocalDateTime localDateTime = ldt.plusYears(2L);
        System.out.println("localDateTime = " + localDateTime);
    }

    /**
     * get方法
     * getYear
     * getMonth
     * getMonthYear
     * getDayOfMonth
     */
    @Test
    public void test4(){
        LocalDateTime now = LocalDateTime.now();
        System.out.println("now.getYear() = " + now.getYear());
        System.out.println("now.getDayOfWeek() = " + now.getDayOfWeek());
    }

}
```

##使用Instant

Instant获取时间戳，以Unix元年：1970年1月1日 00:00:00到某个时间之间的毫秒值

默认获取的是UTC时区的时间（世界协调时间）

```java
public class TestInstant {
    // 2. Instant : 时间戳
    @Test
    public void test1(){
        Instant ins1 = Instant.now();
        // UTC时间
        System.out.println("ins1 = " + ins1);
        // 东八区偏移8小时
        OffsetDateTime odt = ins1.atOffset(ZoneOffset.ofHours(8));
        System.out.println("odt = " + odt);
        // 转毫秒
        long l = ins1.toEpochMilli();
        System.out.println("l = " + l);
        // 与Unix元年偏移一秒
        Instant ins2 = Instant.ofEpochSecond(1l);
        System.out.println("ins2 = " + ins2);
    }
}
```

##Duration时间间隔

```java
public class TestDuration {
    // 3. Duration：时间间隔
    @Test
    public void test1() throws InterruptedException {
        Instant ins1 = Instant.now();
        Instant ins2 = ins1.plusSeconds(10000L);
        Duration duration = Duration.between(ins1, ins2);
        System.out.println("duration = " + duration.getSeconds());
        System.out.println("duration = " + duration.toHours());
        System.out.println("duration = " + duration.toMinutes());
        System.out.println("------------------");
        LocalTime ldt1 = LocalTime.now();
        LocalTime ldt2 = ldt1.plusMinutes(3L);
        Duration between = Duration.between(ldt1, ldt2);
        System.out.println("between = " + between.getSeconds());
    }
}
```

##Period日期间隔

```java
public class TestPeriod {

    @Test
    public void test1(){
        LocalDate ld1 = LocalDate.of(2018, 9, 4);
        LocalDate ld2 = LocalDate.now();
        Period period = Period.between(ld1, ld2);
        System.out.println("period = " + period);
        System.out.println("period.getYears() = " + period.getYears());
        System.out.println("period.getMonths() = " + period.getMonths());
        System.out.println("period.getDays() = " + period.getDays());
    }

}
```

注意：这里get方法获取的是直接比较，而不是差总共多少个月，若都是9月但年份不一样，则getMonths()得到的是0；

