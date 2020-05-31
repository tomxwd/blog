---
2020-03-26 08:39:47
---

# Quartz使用

步骤：

1. 导包
2. 定义自己的任务，实现Job接口，在execute方法中写需要执行的代码（jobExecutionContext能获取到当前job相关信息）
3. 创建Scheduler调度器
4. 创建Trigger触发器，在这里定义触发条件、周期等
5. 创建JobDetail，定义组名等信息
6. 把Trigger和JobDetail交给任务调度器Scheduler，启动调度器，停止调度器。

## 导入依赖

```xml
<!-- quartz依赖导入 -->
<dependency>
    <groupId>org.quartz-scheduler</groupId>
    <artifactId>quartz</artifactId>
    <version>2.3.2</version>
</dependency>
```

## 定义Job

```java
/**
 * Job类
 * @author Administrator
 */
public class MyJob implements Job {
    public void execute(JobExecutionContext jobExecutionContext) throws JobExecutionException {
        JobDetail jobDetail = jobExecutionContext.getJobDetail();
        String group = jobDetail.getKey().getGroup();
        String name = jobDetail.getKey().getName();
        String data = jobDetail.getJobDataMap().getString("data");
        System.out.println("-----> job 执行, job组：" + group + " job名：" + name + " job数据：" + data+"------>"+new Date());

    }
}
```

## API测试

```java
public static void main(String[] args) {
    try {
        // 1. 创建Scheduler调度器---核心组件
        Scheduler scheduler = StdSchedulerFactory.getDefaultScheduler();
        // 2. 定义一个Trigger，触发条件类
        Trigger trigger = TriggerBuilder.newTrigger()
            // 定义name/group
            .withIdentity("trigger1", "group1")
            // 一旦加入scheduler，立即生效，即开始时间
            .startNow()
            .withSchedule(SimpleScheduleBuilder.simpleSchedule()
                          // 每隔2秒执行一次
                          .withIntervalInSeconds(1)
                          // 重复次数
                          .repeatForever()
                         )
            // 设置结束时间，不设置则不结束
            .endAt(new GregorianCalendar(2020, 3, 28, 12, 30, 30).getTime())
            .build();
        // 3. 定义一个JobDetail
        // 定义Job类为HelloQuartz类，这才是真正执行逻辑所在地
        JobDetail job = JobBuilder.newJob(MyJob.class)
            // 定义name/group
            .withIdentity("测试任务名1", "测试组1")
            // 定义属性，存储数据
            .usingJobData("data", "我的job数据")
            .build();
        // 4. 往调度器里添加任务和触发器
        scheduler.scheduleJob(job, trigger);
        // 5. 启动调度器，内部注册的所有触发器开始计时
        scheduler.start();
        Thread.sleep(50000);
        // 6. 关闭调度器
        scheduler.shutdown();
    } catch (SchedulerException | InterruptedException e) {
        e.printStackTrace();
    }
}
```

触发器和job必须一一对应

## 配置

```properties
# 名为：quartz.properties，放在claspath路径下，没有则按默认配置启动
# 指定调度器名称 非实现类
org.quartz.scheduler.instanceName = DefaultQuartzSchedular
# 指定连接池实现类
org.quartz.threadPool.class = org.quartz.simpl.SimpleThreadPool
# 连接池线程数量
org.quartz.threadPool.threadCount = 10
# 线程优先级 默认5
org.quartz.threadPool.threadPriority = 5
# 非持久化job RAMJobStore放在内存里
org.quartz.jobStore.class = org.quartz.simpl.RAMJobStore
```




## 核心类说明

### Scheduler

- 调度器，所有调度都是由控制
- Scheduler就是Quartz的大脑，所有任务由它实施
- Scheduler包含两个重要组件：
  - JobStore：存储运行时信息，包括Trigger，Scheduler，JobDetail，业务锁等
  - ThreadPool：线程池，Quartz有自己的线程池实现，所有任务由线程池执行

### SchedulerFactory

顾名思义这个是用来创建Scheduler的工厂类，有两个具体实现：

- DirectSchedulerFactory：可以用来在代码里定制自己的Scheduler参数
- StdSchedulerFactory：直接读取classpath下的quartz.properties（不存在就用默认值），来实例化Scheduler

一般来说，用StdSchedulerFactory就足够了

SchedulerFactory本身是支持创建RMI stub的，可以用来管理远程的Scheduler，功能与本地一样。



# Trigger（重点）

这里是Trigger中的withSchedule()里的参数

## SimpleSchedule

指定某一个时间开始，以一定时间间隔（单位是毫秒）执行任务。

适合类似于9:00开始，每隔一小时执行一次

属性：

- repeatInterval：重复间隔。
- repeatCount：重复次数，实际是repeatCount+1，因为在startNow的时候会执行一次。

示例：

```java
SimpleScheduleBuilder.simpleSchedule()
    .withIntervalInSeconds(10)// 每隔10秒执行一次
    .repeatForever()//永远执行
    .build();
```

```java
SimpleScheduleBuilder.simpleSchedule()
    .withIntervalInMinutes(3)// 每隔3分钟执行一次
    .withRepeatCount(3)// 重复执行3次，实际上是3+1
    .build();
```

## CalendarIntervalSchedule

类似SimpleTrigger，指定某一个事件开始，以一定时间间隔执行任务。

但是不同的是，SimpleTrigger指定的事件间隔为毫秒，没办法指定每隔一个月执行一次（因为每个月的时间间隔不是固定的值），而CalendarIntervalTrigger支持的间隔单位有秒、分钟、小时、天、月、年、星期。

示例：

```java
CalendarIntervalScheduleBuilder.calendarIntervalSchedule()
    .withIntervalInDays(2)// 每两天执行一次
    .build();
```

```java
CalendarIntervalScheduleBuilder.calendarIntervalSchedule()
    .withIntervalInWeeks(1)// 每周执行一次
    .build();
```

## DailyTimeIntervalTrigger

指定每天的某个时间段内，以一定的时间间隔执任务，并且可以支持指定星期。

适合类似于：指定每天9:00到18:00，每隔70秒执行一次，并且是在周一到周五执行。

属性：

- startTimeOfDay：每天开始时间
- endTimeOfDay：每天结束时间
- daysOfWeek：需要执行的星期
- interval：执行间隔
- intervalUnit：执行间隔的单位（秒，分钟，小时，天，月，年，星期）
- repeatCount：重复次数

示例：

```java
DailyTimeIntervalScheduleBuilder.dailyTimeIntervalSchedule()
    .startingDailyAt(TimeOfDay.hourAndMinuteOfDay(9, 0))// 每天9点开始
    .endingDailyAt(TimeOfDay.hourAndMinuteOfDay(18, 0))// 每天18点接洪水
    .onDaysOfTheWeek(MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY)// 周一到周五执行
    .withIntervalInHours(1)// 间隔1小时执行一次
    .withRepeatCount(100)// 最多重复100次（实际执行101次）
    .build();
```

```java
DailyTimeIntervalScheduleBuilder.dailyTimeIntervalSchedule()
    .startingDailyAt(TimeOfDay.hourAndMinuteOfDay(10, 0)) // 每天10点
    .endingDailyAfterCount(10)// 执行10次
    .onDaysOfTheWeek(MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY)// 周一到周五执行
    .withIntervalInHours(1)// 每间隔1小时执行1次
    .build();
```

## CronTrigger(重点)

适合更复杂的任务，支持类似于Linux Cron的语法（并且更强大），基本上覆盖了上面3个Trigger的绝大部分能力（不是全部），语法也不好理解；

适合的任务类似于：每天0:00，9:00，18:00各执行一次。

属性只有一个：Cron表达式，足够复杂

示例：

```java
// 每天10点到12点，每隔2分钟触发一次
CronScheduleBuilder.cronSchedule("0 0/2 10-12 * * ?").build();

// 每周一，9:30执行一次
CronScheduleBuilder.cronSchedule("0 30 9 ? * MON").build();

// 等同于 0 30 9 ? * MON
CronScheduleBuilder.weeklyOnDayAndHourAndMinute(MONDAY, 9, 30).build();
```



### Cron表达式

> [cron表达式验证网站](http://www.bejson.com/othertools/cronvalidate/)

| 位置 | 时间域     | 允许值                                                       | 特殊值            |
| ---- | ---------- | ------------------------------------------------------------ | ----------------- |
| 1    | 秒         | 0-59                                                         | , - * /           |
| 2    | 分钟       | 0-59                                                         | , - * /           |
| 3    | 小时       | 0-23                                                         | , - * /           |
| 4    | 日期       | 1-31                                                         | , - * ? / L W C   |
| 5    | 月份       | 1-12 或 JAN-DEC                                              | , - * /           |
| 6    | 星期       | 1-7 或 SUN-SAT（注意，这里是从周日开始计算的，所以1是SUN周日） | , - * ? / L W C # |
| 7    | 年份(可选) | 1970-2099                                                    | , - * /           |



```
*：代表所有可能的值
-：指定范围
,：列出枚举  例如在分钟里，"5,15"表示5分钟和20分钟触发
/：指定增量  例如在分钟里，"3/15"表示从3分钟开始，没隔15分钟执行一次
?：表示没有具体的值，使用?要注意冲突
L：表示last，例如星期中表示7或SAT，月份中表示最后一天31或30，6L表示这个月倒数第6天，FRIL表示这个月的最后一个星期五
W：只能用在月份中，表示最接近指定天的工作日
#：只能用在星期中，表示这个月的第几个周几，例如6#3表示这个月的第3个周五
```



1. \*：表示匹配该域的任意值。假如在Minutes域使用\*, 即表示每分钟都会触发事件。
2. ?：只能用在DayofMonth和DayofWeek两个域。它也匹配域的任意值，但实际不会。因为DayofMonth和DayofWeek会相互影响。例如想在每月的20日触发调度，不管20日到底是星期几，则只能使用如下写法： 13 13 15 20 * ?, 其中最后一位只能用？，而不能使用\*，如果使用\*表示不管星期几都会触发，实际上并不是这样。
3. -：表示范围。例如在Minutes域使用5-20，表示从5分到20分钟每分钟触发一次 
4. /：表示起始时间开始触发，然后每隔固定时间触发一次。例如在Minutes域使用5/20,则意味着5分钟触发一次，而25，45等分别触发一次. 
5. ,：表示列出枚举值。例如：在Minutes域使用5,20，则意味着在5和20分每分钟触发一次。 
6. L：表示最后，只能出现在DayofWeek和DayofMonth域。如果在DayofWeek中只有L（表示最后的最后），则表示周六等同于7，如果在DayofWeek域使用5L,意味着在最后的一个星期四触发。 
7. W：表示有效工作日(周一到周五),只能出现在DayofMonth域，系统将在离指定日期的最近的有效工作日触发事件。例如：在 DayofMonth使用5W，如果5日是星期六，则将在最近的工作日：星期五，即4日触发。如果5日是星期天，则在6日(周一)触发；如果5日在星期一到星期五中的一天，则就在5日触发。另外一点，W的最近寻找不会跨过月份 。
8. LW：这两个字符可以连用，表示在某个月最后一个工作日，即最后一个星期五。
9. #：用于确定每个月第几个星期几，只能出现在DayofMonth域。例如在4#2，表示某月的第二个星期三。



示例：

```
0 * * * * ? 每1分钟触发一次
0 0 * * * ? 每天每1小时触发一次
0 0 10 * * ? 每天10点触发一次
0 * 14 * * ? 在每天下午2点到下午2:59期间的每1分钟触发
0 30 9 1 * ? 每月1号上午9点半
0 15 10 15 * ? 每月15日上午10:15触发
*/5 * * * * ? 每隔5秒执行一次
0 */1 * * * ? 每隔1分钟执行一次
0 0 5-15 * * ? 每天5-15点整点触发
0 0/3 * * * ? 每三分钟触发一次
0 0 0 1 * ?  每月1号凌晨执行一次
```



# Job并发（重点）

job是有可能并发执行的，比如一个任务要执行10秒，而调度算法是每秒出发1次，那么多个任务就会被并发执行。

有时候我们并不希望任务是并发执行的，比如任务是要去获取数据库中没有发送邮件的名单，如果是并发执行，就需要一个数据库锁去避免一个数据被多次处理，这时候一个@DisallowConcurrentExecution解决这个问题。

示例：

```java
@DisallowConcurrentExecution
public class MyJob2 implements Job {

    @SneakyThrows
    @Override
    public void execute(JobExecutionContext context) throws JobExecutionException {
        System.out.println(Thread.currentThread().getName() + " 进来了");
        TimeUnit.SECONDS.sleep(3);
        System.out.println(Thread.currentThread().getName() + " 完成了");
    }
}
```

**注意：**

@DisallowConcurrentExecution是对JobDetail实例生效，也就是如果你定义了两个JobDetail，还是会并发执行的。



# Spring整合Quartz（重点）

## 导包

pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>top.tomxwd.test</groupId>
    <artifactId>qianfeng-quartz-spring</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>5.2.5.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-tx</artifactId>
            <version>5.2.5.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context-support</artifactId>
            <version>5.2.5.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
            <version>5.2.5.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.quartz-scheduler</groupId>
            <artifactId>quartz</artifactId>
            <version>2.3.2</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.13</version>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.1.21</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>5.2.5.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.39</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.1</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                    <encoding>utf-8</encoding>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>
```

注意需要引入spring的context-support包，里面有对quartz的支持。导入tx、jdbc、druid是为了后面的持久化行为。

## 配置

步骤：

1. 定义工作任务的Job
2. 定义触发器Trigger，并将触发器和Job绑定
3. 定义调度器，并将Trigger注册到Scheduler

```java
@Configuration
public class QuartzConfig {

    /**
     * 定义两个Job
     *
     * @return
     */
    @Bean
    public JobDetailFactoryBean myJob1() {
        JobDetailFactoryBean jobDetailFactoryBean = new JobDetailFactoryBean();
        // 指定job的名称
        jobDetailFactoryBean.setName("jobName1");
        // 指定job的分组
        jobDetailFactoryBean.setGroup("jobGroup");
        // 指定具体的job类
        jobDetailFactoryBean.setJobClass(MyJob.class);
        // 【可选】如果为false，那么在没有获得的Trigger指向他的时候会被删除
        jobDetailFactoryBean.setDurability(true);
        /*
            【可选】配置：
            指定Spring容器的key，如果不设定，job中的jobmap会取不到spring容器
            因为jobDetailFactoryBean实现了ApplicationContextWare接口，
            则其中的setApplicationContext方法会得到当前的工厂对象
            且将工厂对象存在了类中的一个属性applicationContext中

            源码如下：
            getJobDataMap().put(this.applicationContextJobDataKey,this.applicationContext);
            则在Job的jobmap中可以获得工厂对象，如果需要可以使用
            (ApplicationContext)jobDataMap.get("applicationContext1996")
            获取容器对象
         */
        jobDetailFactoryBean.setApplicationContextJobDataKey("applicationContext1996");
        return jobDetailFactoryBean;
    }

    @Bean
    public JobDetailFactoryBean myJob2() {
        JobDetailFactoryBean jobDetailFactoryBean = new JobDetailFactoryBean();
        jobDetailFactoryBean.setName("jobName2");
        jobDetailFactoryBean.setGroup("jobGroup");
        jobDetailFactoryBean.setJobClass(MyJob.class);
        jobDetailFactoryBean.setDurability(true);
        jobDetailFactoryBean.setApplicationContextJobDataKey("applicationContext1996");
        return jobDetailFactoryBean;
    }

    @Bean
    public CronTriggerFactoryBean myCronTrigger1() {
        CronTriggerFactoryBean cronTrigger = new CronTriggerFactoryBean();
        // 指定Trigger的名称
        cronTrigger.setName("triggerName1");
        // 指定Trigger所在的group
        cronTrigger.setGroup("triggerGroup");
        // 指定Trigger绑定的Job
        cronTrigger.setJobDetail(myJob1().getObject());
        // 指定Cron表达式，这里是2s执行一次
        cronTrigger.setCronExpression("*/1 * * * * ?");
        return cronTrigger;
    }

    @Bean
    public CronTriggerFactoryBean myCronTrigger2() {
        CronTriggerFactoryBean cronTrigger = new CronTriggerFactoryBean();
        cronTrigger.setName("triggerName2");
        cronTrigger.setGroup("triggerGroup");
        cronTrigger.setJobDetail(myJob2().getObject());
        cronTrigger.setCronExpression("*/1 * * * * ?");
        return cronTrigger;
    }

    @Bean
    public SchedulerFactoryBean myScheduler() {
        SchedulerFactoryBean scheduler = new SchedulerFactoryBean();
        scheduler.setTriggers(myCronTrigger1().getObject(), myCronTrigger2().getObject());
        Properties properties = new Properties();
        try {
            properties.load(this.getClass().getClassLoader().getResourceAsStream("quartz.properties"));
        } catch (IOException e) {
            System.out.println("读取quartz配置文件失败");
        }
//        scheduler.setConfigLocation(new Resource);
        scheduler.setQuartzProperties(properties);
        return scheduler;
    }

}
```



测试：

```java
public class QuartzTest {

    @Test
    public void test1() throws InterruptedException, SchedulerException {
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(QuartzConfig.class);
        Scheduler scheduler = applicationContext.getBean("myScheduler", Scheduler.class);
        TimeUnit.SECONDS.sleep(2);
        // 暂停任务1
        System.out.println("暂停任务1");
        scheduler.pauseJob(new JobKey("jobName1", "jobGroup"));
        TimeUnit.SECONDS.sleep(5);
        // 重启任务1
        System.out.println("重启任务1");
        scheduler.resumeJob(new JobKey("jobName1", "jobGroup"));
        TimeUnit.SECONDS.sleep(20);
    }

    @Test
    public void test2() throws SchedulerException, InterruptedException {
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(QuartzConfig.class);
        Scheduler myScheduler = applicationContext.getBean("myScheduler", Scheduler.class);
        TimeUnit.SECONDS.sleep(2);
        System.out.println("暂定触发器1");
        myScheduler.pauseTrigger(new TriggerKey("triggerName1","triggerGroup"));
        TimeUnit.SECONDS.sleep(2);
        System.out.println("重启触发器1");
        myScheduler.resumeTrigger(TriggerKey.triggerKey("triggerName1","triggerGroup"));
        TimeUnit.SECONDS.sleep(20);
    }

}
```

可以获取到调度器Scheduler之后，进行一些暂停，重启job、Trigger操作，可以写在controller来进行控制。

- pause：暂停（这时候只是暂停job的执行，但是任务还是一直在累积）
- unschedule：移除
- delete：删除(最好暂停->移除->删除)
- resume：重启

这里可以传入组信息，job信息，name信息，Trigger信息都可以。

还有操作All方法，还可以addjob等。



# 持久化

建表语句在quartz\docs\dbTables文件夹中可以找到，到[官网](http://www.quartz-scheduler.org/downloads/)下载他们的tar包

**注意：需要执行的sql文件只有一个，尾缀为mysql_innodb.sql**

## 建表

```sql
#
# In your Quartz properties file, you'll need to set 
# org.quartz.jobStore.driverDelegateClass = org.quartz.impl.jdbcjobstore.StdJDBCDelegate
#
#
# By: Ron Cordell - roncordell
#  I didn't see this anywhere, so I thought I'd post it here. This is the script from Quartz to create the tables in a MySQL database, modified to use INNODB instead of MYISAM.

DROP TABLE IF EXISTS QRTZ_FIRED_TRIGGERS;
DROP TABLE IF EXISTS QRTZ_PAUSED_TRIGGER_GRPS;
DROP TABLE IF EXISTS QRTZ_SCHEDULER_STATE;
DROP TABLE IF EXISTS QRTZ_LOCKS;
DROP TABLE IF EXISTS QRTZ_SIMPLE_TRIGGERS;
DROP TABLE IF EXISTS QRTZ_SIMPROP_TRIGGERS;
DROP TABLE IF EXISTS QRTZ_CRON_TRIGGERS;
DROP TABLE IF EXISTS QRTZ_BLOB_TRIGGERS;
DROP TABLE IF EXISTS QRTZ_TRIGGERS;
DROP TABLE IF EXISTS QRTZ_JOB_DETAILS;
DROP TABLE IF EXISTS QRTZ_CALENDARS;

CREATE TABLE QRTZ_JOB_DETAILS(
SCHED_NAME VARCHAR(120) NOT NULL,
JOB_NAME VARCHAR(200) NOT NULL,
JOB_GROUP VARCHAR(200) NOT NULL,
DESCRIPTION VARCHAR(250) NULL,
JOB_CLASS_NAME VARCHAR(250) NOT NULL,
IS_DURABLE VARCHAR(1) NOT NULL,
IS_NONCONCURRENT VARCHAR(1) NOT NULL,
IS_UPDATE_DATA VARCHAR(1) NOT NULL,
REQUESTS_RECOVERY VARCHAR(1) NOT NULL,
JOB_DATA BLOB NULL,
PRIMARY KEY (SCHED_NAME,JOB_NAME,JOB_GROUP))
ENGINE=InnoDB;

CREATE TABLE QRTZ_TRIGGERS (
SCHED_NAME VARCHAR(120) NOT NULL,
TRIGGER_NAME VARCHAR(200) NOT NULL,
TRIGGER_GROUP VARCHAR(200) NOT NULL,
JOB_NAME VARCHAR(200) NOT NULL,
JOB_GROUP VARCHAR(200) NOT NULL,
DESCRIPTION VARCHAR(250) NULL,
NEXT_FIRE_TIME BIGINT(13) NULL,
PREV_FIRE_TIME BIGINT(13) NULL,
PRIORITY INTEGER NULL,
TRIGGER_STATE VARCHAR(16) NOT NULL,
TRIGGER_TYPE VARCHAR(8) NOT NULL,
START_TIME BIGINT(13) NOT NULL,
END_TIME BIGINT(13) NULL,
CALENDAR_NAME VARCHAR(200) NULL,
MISFIRE_INSTR SMALLINT(2) NULL,
JOB_DATA BLOB NULL,
PRIMARY KEY (SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP),
FOREIGN KEY (SCHED_NAME,JOB_NAME,JOB_GROUP)
REFERENCES QRTZ_JOB_DETAILS(SCHED_NAME,JOB_NAME,JOB_GROUP))
ENGINE=InnoDB;

CREATE TABLE QRTZ_SIMPLE_TRIGGERS (
SCHED_NAME VARCHAR(120) NOT NULL,
TRIGGER_NAME VARCHAR(200) NOT NULL,
TRIGGER_GROUP VARCHAR(200) NOT NULL,
REPEAT_COUNT BIGINT(7) NOT NULL,
REPEAT_INTERVAL BIGINT(12) NOT NULL,
TIMES_TRIGGERED BIGINT(10) NOT NULL,
PRIMARY KEY (SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP),
FOREIGN KEY (SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP)
REFERENCES QRTZ_TRIGGERS(SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP))
ENGINE=InnoDB;

CREATE TABLE QRTZ_CRON_TRIGGERS (
SCHED_NAME VARCHAR(120) NOT NULL,
TRIGGER_NAME VARCHAR(200) NOT NULL,
TRIGGER_GROUP VARCHAR(200) NOT NULL,
CRON_EXPRESSION VARCHAR(120) NOT NULL,
TIME_ZONE_ID VARCHAR(80),
PRIMARY KEY (SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP),
FOREIGN KEY (SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP)
REFERENCES QRTZ_TRIGGERS(SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP))
ENGINE=InnoDB;

CREATE TABLE QRTZ_SIMPROP_TRIGGERS
  (          
    SCHED_NAME VARCHAR(120) NOT NULL,
    TRIGGER_NAME VARCHAR(200) NOT NULL,
    TRIGGER_GROUP VARCHAR(200) NOT NULL,
    STR_PROP_1 VARCHAR(512) NULL,
    STR_PROP_2 VARCHAR(512) NULL,
    STR_PROP_3 VARCHAR(512) NULL,
    INT_PROP_1 INT NULL,
    INT_PROP_2 INT NULL,
    LONG_PROP_1 BIGINT NULL,
    LONG_PROP_2 BIGINT NULL,
    DEC_PROP_1 NUMERIC(13,4) NULL,
    DEC_PROP_2 NUMERIC(13,4) NULL,
    BOOL_PROP_1 VARCHAR(1) NULL,
    BOOL_PROP_2 VARCHAR(1) NULL,
    PRIMARY KEY (SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP),
    FOREIGN KEY (SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP) 
    REFERENCES QRTZ_TRIGGERS(SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP))
ENGINE=InnoDB;

CREATE TABLE QRTZ_BLOB_TRIGGERS (
SCHED_NAME VARCHAR(120) NOT NULL,
TRIGGER_NAME VARCHAR(200) NOT NULL,
TRIGGER_GROUP VARCHAR(200) NOT NULL,
BLOB_DATA BLOB NULL,
PRIMARY KEY (SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP),
INDEX (SCHED_NAME,TRIGGER_NAME, TRIGGER_GROUP),
FOREIGN KEY (SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP)
REFERENCES QRTZ_TRIGGERS(SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP))
ENGINE=InnoDB;

CREATE TABLE QRTZ_CALENDARS (
SCHED_NAME VARCHAR(120) NOT NULL,
CALENDAR_NAME VARCHAR(200) NOT NULL,
CALENDAR BLOB NOT NULL,
PRIMARY KEY (SCHED_NAME,CALENDAR_NAME))
ENGINE=InnoDB;

CREATE TABLE QRTZ_PAUSED_TRIGGER_GRPS (
SCHED_NAME VARCHAR(120) NOT NULL,
TRIGGER_GROUP VARCHAR(200) NOT NULL,
PRIMARY KEY (SCHED_NAME,TRIGGER_GROUP))
ENGINE=InnoDB;

CREATE TABLE QRTZ_FIRED_TRIGGERS (
SCHED_NAME VARCHAR(120) NOT NULL,
ENTRY_ID VARCHAR(95) NOT NULL,
TRIGGER_NAME VARCHAR(200) NOT NULL,
TRIGGER_GROUP VARCHAR(200) NOT NULL,
INSTANCE_NAME VARCHAR(200) NOT NULL,
FIRED_TIME BIGINT(13) NOT NULL,
SCHED_TIME BIGINT(13) NOT NULL,
PRIORITY INTEGER NOT NULL,
STATE VARCHAR(16) NOT NULL,
JOB_NAME VARCHAR(200) NULL,
JOB_GROUP VARCHAR(200) NULL,
IS_NONCONCURRENT VARCHAR(1) NULL,
REQUESTS_RECOVERY VARCHAR(1) NULL,
PRIMARY KEY (SCHED_NAME,ENTRY_ID))
ENGINE=InnoDB;

CREATE TABLE QRTZ_SCHEDULER_STATE (
SCHED_NAME VARCHAR(120) NOT NULL,
INSTANCE_NAME VARCHAR(200) NOT NULL,
LAST_CHECKIN_TIME BIGINT(13) NOT NULL,
CHECKIN_INTERVAL BIGINT(13) NOT NULL,
PRIMARY KEY (SCHED_NAME,INSTANCE_NAME))
ENGINE=InnoDB;

CREATE TABLE QRTZ_LOCKS (
SCHED_NAME VARCHAR(120) NOT NULL,
LOCK_NAME VARCHAR(40) NOT NULL,
PRIMARY KEY (SCHED_NAME,LOCK_NAME))
ENGINE=InnoDB;

CREATE INDEX IDX_QRTZ_J_REQ_RECOVERY ON QRTZ_JOB_DETAILS(SCHED_NAME,REQUESTS_RECOVERY);
CREATE INDEX IDX_QRTZ_J_GRP ON QRTZ_JOB_DETAILS(SCHED_NAME,JOB_GROUP);

CREATE INDEX IDX_QRTZ_T_J ON QRTZ_TRIGGERS(SCHED_NAME,JOB_NAME,JOB_GROUP);
CREATE INDEX IDX_QRTZ_T_JG ON QRTZ_TRIGGERS(SCHED_NAME,JOB_GROUP);
CREATE INDEX IDX_QRTZ_T_C ON QRTZ_TRIGGERS(SCHED_NAME,CALENDAR_NAME);
CREATE INDEX IDX_QRTZ_T_G ON QRTZ_TRIGGERS(SCHED_NAME,TRIGGER_GROUP);
CREATE INDEX IDX_QRTZ_T_STATE ON QRTZ_TRIGGERS(SCHED_NAME,TRIGGER_STATE);
CREATE INDEX IDX_QRTZ_T_N_STATE ON QRTZ_TRIGGERS(SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP,TRIGGER_STATE);
CREATE INDEX IDX_QRTZ_T_N_G_STATE ON QRTZ_TRIGGERS(SCHED_NAME,TRIGGER_GROUP,TRIGGER_STATE);
CREATE INDEX IDX_QRTZ_T_NEXT_FIRE_TIME ON QRTZ_TRIGGERS(SCHED_NAME,NEXT_FIRE_TIME);
CREATE INDEX IDX_QRTZ_T_NFT_ST ON QRTZ_TRIGGERS(SCHED_NAME,TRIGGER_STATE,NEXT_FIRE_TIME);
CREATE INDEX IDX_QRTZ_T_NFT_MISFIRE ON QRTZ_TRIGGERS(SCHED_NAME,MISFIRE_INSTR,NEXT_FIRE_TIME);
CREATE INDEX IDX_QRTZ_T_NFT_ST_MISFIRE ON QRTZ_TRIGGERS(SCHED_NAME,MISFIRE_INSTR,NEXT_FIRE_TIME,TRIGGER_STATE);
CREATE INDEX IDX_QRTZ_T_NFT_ST_MISFIRE_GRP ON QRTZ_TRIGGERS(SCHED_NAME,MISFIRE_INSTR,NEXT_FIRE_TIME,TRIGGER_GROUP,TRIGGER_STATE);

CREATE INDEX IDX_QRTZ_FT_TRIG_INST_NAME ON QRTZ_FIRED_TRIGGERS(SCHED_NAME,INSTANCE_NAME);
CREATE INDEX IDX_QRTZ_FT_INST_JOB_REQ_RCVRY ON QRTZ_FIRED_TRIGGERS(SCHED_NAME,INSTANCE_NAME,REQUESTS_RECOVERY);
CREATE INDEX IDX_QRTZ_FT_J_G ON QRTZ_FIRED_TRIGGERS(SCHED_NAME,JOB_NAME,JOB_GROUP);
CREATE INDEX IDX_QRTZ_FT_JG ON QRTZ_FIRED_TRIGGERS(SCHED_NAME,JOB_GROUP);
CREATE INDEX IDX_QRTZ_FT_T_G ON QRTZ_FIRED_TRIGGERS(SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP);
CREATE INDEX IDX_QRTZ_FT_TG ON QRTZ_FIRED_TRIGGERS(SCHED_NAME,TRIGGER_GROUP);

commit; 
```



## 配置

Quartz-spring配置文件：

```java

```



quartz.properties：

```properties
# 名为：quartz.properties，放在claspath路径下，没有则按默认配置启动
# 指定调度器名称 非实现类
org.quartz.scheduler.instanceName = DefaultQuartzSchedular
# 指定连接池实现类
org.quartz.threadPool.class = org.quartz.simpl.SimpleThreadPool
# 连接池线程数量
org.quartz.threadPool.threadCount = 18
# 线程优先级 默认5
org.quartz.threadPool.threadPriority = 5
# 非持久化job RAMJobStore放在内存里
# org.quartz.jobStore.class = org.quartz.simpl.RAMJobStore
# 持久化
org.quartz.jobStore.class = org.quartz.impl.jdbcjobstore.JobStoreTX
# quartz表的前缀
org.quartz.jobSotre.tablePrefix = qrtz_
```















