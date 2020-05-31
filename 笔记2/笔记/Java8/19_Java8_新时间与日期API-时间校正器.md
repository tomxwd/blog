---
2019-09-04 17:20:27

---

#

##日期的操作

- TemporalAdjuster：时间校正器。
- TemporalAdjusters：该类通过静态方法提供了大量的常用TemporalAdjuster的实现。

有时我们可能需要获取例如：

将日期调整到“下个周日”等操作；

```java
LocalDate nextSunday = LocalDate.now().with(
    TemporalAdjusters.next(DayOfWeek.SUNDAY)
);
```

