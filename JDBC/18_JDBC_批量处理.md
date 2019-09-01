---
title: 18_JDBC_批量处理
date: 2019-08-23 14:12:02
tags: 
 - Java
categories:
 - JDBC
---

# 18_JDBC_批量处理

## 批量处理JDBC语句提高处理速度

- 当需要成批插入或者更新记录的时候，可以采用Java的批量更新机制，这一机制允许多条语句一次性提交给数据库批量处理。通常情况下比单独提交处理更有效率；
- JDBC的批量处理语句包括下面两个方法：
  - addBatch(String)：添加需要批量处理的SQL语句或是参数；
  - executeBatch()；执行批量处理语句；
- 通常我们会遇到两种批量执行SQL语句的情况：
  - 多条SQL语句的批量处理；
  - 一个SQL语句的批量传参；



## 测试

这里最好用Oracle进行测试，因为mysql的速度差别不大，看不出效果。

创建好测试用的表，然后编写Java程序：

这里用System.currentTimeMillis();来反映时长。

1. 使用Statement进行插入，插入10万条数据；（39567ms）

2. 使用PreparedStatement进行插入，插入10万条数据；（9819ms）

3. 积攒着，一次性进行插入，调用`preparedStatement.addBatch();`，也不可以一直积攒着，积攒到一定就执行一次插入，比如每300条执行一次插入:

   ```java
   // 积攒SQL
   preparedStatement.addBatch();
   // 当积攒到一定程度就执行一次，比如300条，并且清空之前积攒的
   if((i+1)%300==0){
       preparedStatement.executeBatch();
       preparedStatement.clearBatch();
   }
   // 最后要清理一次，因为可能最后一场并没有300条整
   if(100000%300!=0){
       preparedStatement.executeBatch();
       preparedStatement.clearBatch();
   }
   ```

   （569）

注意：mysql不起作用。

