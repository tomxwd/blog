---
title: 01_Log4j配置详解
date: 2019-09-21 18:46:40
tags: 
 - Java
 - Log4j
categories:
 - Log4j
---

# 01_Log4j配置详解



## rootLogger根配置

log4j.rootLogger = [level],appenderName,appenderName,......

例如：

log4j.rootLogger = INFO,Console,File

把info即以上的等级信息输出到控制台以及指定文件中去。



## log4j日志等级

- OFF
- FATAL
- ERROR
- WARN
- INFO
- DEBUG
- ALL



## log4j appender输出类型配置

1. org.apache.log4j.ConsoleAppender（控制台）
2. org.apache.log4j.FileAppender（指定文件）
   - File：指定输出文件地址
3. org.apache.log4j.DailyRollingFileAppender（每天产生一个日志文件）
4. org.apache.log4j.RollingFileAppender（文件大小达到指定尺寸的时候产生一个新的文件）
   - MaxFileSize：指定日志文件的最大尺寸
   - MaxBackupIndex是日志文件的个数，如果超过了就覆盖，也是考虑到磁盘的容量问题
5. org.apache.log4j.WriterAppender（将日志信息以流格式发送到任意指定地方）

一般使用1、3、4，这三种来开发。



## log4j layout日志信息格式

1. org.apache.log4j.HTMLLayout（以HTML表格形式布局）
2. **org.apache.log4j.PatternLayout（可以灵活的指定布局模式）**【一般都是用这个】
3. org.apache.log4j.SimpleLayout（包含日志信息的级别和信息字符串）
4. org.apache.log4j.TTCCLayout（包含日志产生的时间、线程、类别等信息）



PatternLayout：

有个ConversionPattern属性，灵活配置输出属性：

| 属性 | 描述                                                         |
| ---- | ------------------------------------------------------------ |
| %m   | 输出代码中指定的信息                                         |
| %M   | 输出打印该条日志的方法名                                     |
| %p   | 输出优先级，即DEBUG、INFO、WARN、ERROR、FATAL                |
| %r   | 输出自应用启动到输出该log信息耗费的毫秒数                    |
| %c   | 输出所属的类目，通常就是所在类的全名                         |
| %t   | 输出产生该日志事件的线程名                                   |
| %n   | 输出一个回车换行符，Windows平台为"rn"，Unix平台为"n"         |
| %d   | 输出日志时间点的日期或事件，默认格式为ISO8601，也可以在后面指定格式<br />比如：%d{yyyy-MM-dd HH:mm:ss,SSS}，输出类似：2019-09-21 20:10:28,578; |
| %l   | 输出日志事件的发生位置，及在代码中的行数                     |

在%之后加上-5，即是指定为占用多少个位置，比如%-5p，那么INFO也会占5个位置，保持跟其他的对齐



## log4j Threshold属性指定输出等级

有时候我们需要把一些报错ERROR日志单独存到指定文件，这时候，Threshold属性就派上用场了;

Threshold属性可以指定日志登记

Log4j根据日志信息的重要程度，分OFF、FATAL、ERROR、WARN、INFO、DEBUG、ALL

比如我们指定某个appender的Threshold为WARN，那么这个appender输出的日志信息就是WARN以及以上的级别信息。

假如我们指定的是ERROR，那这个就输出ERROR或者FATAL日志信息；

当然这里有个前提，就是rootLogger里配置的level小于这里的Threshold等级，否则无效，还是按照总的rootLogger里的level来输出，一般我们实际使用的话，rootLogger里配置DEBUG，然后某个文件专门存储ERROR日志，就配置Threshold为ERROR，这个就是我们的最佳实践。



## log4j Append属性指定是否追加内容

默认是追加到文件后面的，Append属性默认为true

假如我们需要覆盖前面的内容就需要改为false

log4j.appender.FIle.Append=false

一般默认即可，不能对日志文件进行覆盖

