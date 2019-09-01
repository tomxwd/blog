---
title: 06_JDBC_以面向对象的方式编写JDBC程序
date: 2019-08-22 08:14:29
tags: 
 - Java
categories:
 - JDBC
---

# 06_JDBC_以面向对象的方式编写JDBC程序

## 准备环境

创建数据库表examstudent：

| 字段名      | 说明       | 类型        |
| ----------- | ---------- | ----------- |
| FlowID      | 流水号     | int         |
| Type        | 四级/六级  | int         |
| IDCard      | 身份证号码 | varchar(18) |
| ExamCard    | 准考证号码 | varchar(15) |
| StudentName | 学生姓名   | varchar(20) |
| Location    | 区域       | varchar(20) |
| Grade       | 成绩       | int         |

创建表语句：

```sql
CREATE TABLE `examstudent` (
  `FlowID` int(11) NOT NULL AUTO_INCREMENT COMMENT '流水号',
  `Type` int(11) DEFAULT NULL COMMENT '四级/六级',
  `IDCard` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci DEFAULT NULL COMMENT '身份证号码',
  `ExamCard` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci DEFAULT NULL COMMENT '准考证号码',
  `StudentName` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci DEFAULT NULL COMMENT '学生姓名',
  `Location` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci DEFAULT NULL COMMENT '区域',
  `Grade` int(11) DEFAULT NULL COMMENT '成绩',
  PRIMARY KEY (`FlowID`)
) ENGINE=InnoDB AUTO_INCREMENT=11 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

插入一条测试数据：

| FlowID | Type | IDCard             | ExamCard        | StudentName | Location | Grade |
| ------ | ---- | ------------------ | --------------- | ----------- | -------- | ----- |
| 1      | 4    | 412824195263214584 | 200523164754000 | 张峰        | 郑州     | 85    |
| 2      | 4    | 222224195263214584 | 200523164754001 | 孙朋        | 大连     | 56    |
| 3      | 6    | 342824195263214584 | 200523164754002 | 刘明        | 沈阳     | 72    |
| 4      | 6    | 100824195263214584 | 200523164754003 | 赵虎        | 哈尔滨   | 95    |
| 5      | 4    | 454524195263214584 | 200523164754004 | 杨丽        | 北京     | 64    |
| 6      | 4    | 854524195263214584 | 200523164754005 | 王小红      | 太原     | 60    |

插入sql语句：

```sql
INSERT INTO `test`.`examstudent` (`FlowID`, `Type`, `IDCard`, `ExamCard`, `StudentName`, `Location`, `Grade`) VALUES ('10', '0', '12312332412', '1333232', 'tom', '深圳', '380');
```

抽取工具类JDBCTools：

```java
public class JDBCTools {

    /**
     * 执行SQL的方法（insert、update、delete）
     * @param sql
     */
    public static void update(String sql){
        Connection connection = null;
        Statement statement = null;
        try {
            // 1. 获取数据库连接
            connection = getConnection();
            // 2. 调用connection对象的createStatement()方法获取Statement对象
            statement = connection.createStatement();
            // 3. 准备SQL语句
            // 4. 发送SQL语句：调用Statement对象的executeUpdate(sql)方法
            statement.executeUpdate(sql);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            // 5. 关闭数据库资源：由里向外关闭
            release(statement,connection);
        }
    }

    /**
     * 释放数据库资源
     * @param statement
     * @param connection
     */
    public static void release(Statement statement, Connection connection){
        if(statement!=null){
            try {
                statement.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        if(connection!=null){
            try {
                connection.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }

    /**
     * 释放数据库资源
     * @param resultSet
     * @param statement
     * @param connection
     */
    public static void release(ResultSet resultSet, Statement statement, Connection connection){
        if(resultSet!=null){
            try {
                resultSet.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        release(statement,connection);
    }

    /**
     * 获取数据库连接
     * @return
     * @throws IOException
     * @throws ClassNotFoundException
     * @throws SQLException
     */
    public static Connection getConnection() throws IOException, ClassNotFoundException, SQLException {
        // 0. 读取jdbc.properties配置文件
        /**
         * properties属性文件对应java中的Properties类
         * 可以使用类加载器来加载这个配置文件
         */
        Properties properties = new Properties();
        properties.load(JDBCTools.class.getClassLoader().getResourceAsStream("jdbc.properties"));
        // 1. 准备4个连接需要用到的字符串 driverClass,jdbcUrl,user,password
        String driverClass = properties.getProperty("driverClass");
        String jdbcUrl = properties.getProperty("jdbcUrl");
        String user = properties.getProperty("user");
        String password = properties.getProperty("password");
        // 2. 加载驱动 Class.forName(driverClass)
        Class.forName(driverClass);
        // 3. 调用DriverManager.getConnection(jdbcUrl,user,password)方法获取数据库连接
        return DriverManager.getConnection(jdbcUrl, user, password);
    }

}
```



## 编写CRUD代码

### 需求1

插入一条新的数据：

请输入考生的详细信息

Type:

IDCard:

ExamCard:

StudentName:

Location:

Grade:

信息录入成功！



#### 思路

1. 把参数student对象插入到数据库中，而不是一个一个属性，所以要封装一个Student对象。

2. 新建一个Student对象，对应examstudents数据表

   - 对象属性
     - int flowId;流水号
     - int type;考试的类型
     - String idCard;身份证号
     - String examCard;准考证号
     - String studentName;学生名
     - String location;地址
     - int grade;成绩

   ```java
   public class Student {
   
       private int flowId;//流水号
       private int type;//考试的类型
       private String idCard;//身份证号
       private String examCard;//准考证号
       private String studentName;//学生名
       private String location;//地址
       private int grade;//成绩
   
       public Student() {
       }
   
       public Student(int flowId, int type, String idCard, String examCard, String studentName, String location, int grade) {
           this.flowId = flowId;
           this.type = type;
           this.idCard = idCard;
           this.examCard = examCard;
           this.studentName = studentName;
           this.location = location;
           this.grade = grade;
       }
   
       public int getFlowId() {
           return flowId;
       }
   
       public void setFlowId(int flowId) {
           this.flowId = flowId;
       }
   
       public int getType() {
           return type;
       }
   
       public void setType(int type) {
           this.type = type;
       }
   
       public String getIdCard() {
           return idCard;
       }
   
       public void setIdCard(String idCard) {
           this.idCard = idCard;
       }
   
       public String getExamCard() {
           return examCard;
       }
   
       public void setExamCard(String examCard) {
           this.examCard = examCard;
       }
   
       public String getStudentName() {
           return studentName;
       }
   
       public void setStudentName(String studentName) {
           this.studentName = studentName;
       }
   
       public String getLocation() {
           return location;
       }
   
       public void setLocation(String location) {
           this.location = location;
       }
   
       public int getGrade() {
           return grade;
       }
   
       public void setGrade(int grade) {
           this.grade = grade;
       }
       
       @Override
       public String toString() {
           return "Student{" +
                   "flowId=" + flowId +
                   ", type=" + type +
                   ", idCard='" + idCard + '\'' +
                   ", examCard='" + examCard + '\'' +
                   ", studentName='" + studentName + '\'' +
                   ", location='" + location + '\'' +
                   ", grade=" + grade +
                   '}';
       }
   }
   ```

3. 新建一个方法：void addStudent(Student student)

   ```java
   public void addStudent(Student student){
       // 1. 准备一条sql语句 这里是用对象属性拼接的insert语句
       String sql = "insert into examstudent values("+student.getFlowId()+","+student.getType()+",'"+student.getIdCard() +"','"+student.getExamCard()+"','"+student.getStudentName()+"','"+student.getLocation()+"',"+student.getGrade()+");";
       System.out.println("sql = " + sql);
       // 2. 调用JDBCTools类的update(sql)方法
       JDBCTools.update(sql);
   }
   ```
   

   
4. 编写一个接收控制台输入的方法Student getStudentFromConsole();

   ```java
   /**
    * 从控制台输入学生的信息
    * @return
    */
   private Student getStudentFromConsole() {
       Scanner scanner = new Scanner(System.in);
       Student student = new Student();
       System.out.println("---------------");
       System.out.print("输入FlowId：");
       student.setFlowId(scanner.nextInt());
       System.out.print("输入Type：");
       student.setType(scanner.nextInt());
       System.out.print("输入IdCard：");
       student.setIdCard(scanner.next());
       System.out.print("输入ExamCard：");
       student.setExamCard(scanner.next());
       System.out.print("输入StudentName：");
       student.setStudentName(scanner.next());
       System.out.print("输入Location：");
       student.setLocation(scanner.next());
       System.out.print("输入Grade：");
       student.setGrade(scanner.nextInt());
       return student;
   }
   ```

   

5. 编写测试方法testAddStudent()，接收用户输入的student信息，调用addStudent方法看结果；

   这里由于IDEA用junit测试貌似有冲突，无法接受输入，所以改用main方法进行测试：

   ```java
   public static void main(String[] args) {
       JDBCTest jdbcTest = new JDBCTest();
       Student student = jdbcTest.getStudentFromConsole();
       System.out.println("student = " + student);
       jdbcTest.addStudent(student);
   }
   ```



### 需求2


输入身份证号或准考证号查询学生信息：

![练习1](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/JDBC/06%E7%BB%83%E4%B9%A01.png)



#### 思路

1. 接收用户输入，先得到查询的类型

   ```java
   /**
    * 从控制台读入一个整数，确定查询的类型
    * 1. 身份证
    * 2. 准考证号
    * 其他. 用户重新输入
    * @return
    */
   private int getSearchTypeFromCosole() {
       Scanner scanner = new Scanner(System.in);
       System.out.println("请输入查询类型 1. 用身份证进行查询 2. 用准考证进行查询");
       int type = scanner.nextInt();
       while (type!=1&&type!=2){
           System.out.println("输入有误，请重新输入：");
           type = scanner.nextInt();
       }
       return type;
   }
   ```

   

2. 接收用户输入，获取学生信息

   ```java
   /**
    * 具体查询学生信息，返回一个Student对象，若不存在，则返回null
    * @param searchType
    * @return
    */
   private Student searchStudent(int searchType) {
       Scanner scanner = new Scanner(System.in);
       String sql = "Select flowid,type,idcard,examcard,studentname,location,grade from examstudent where ";
       // 1. 根据输入的searchType，提示用户输入信息
       String number = "";
       // 2. 根据searchType确定SQL
       if(searchType==1){
           // 1.1 若type为1则提示输入身份证
           System.out.print("请输入身份证号：");
           number = scanner.next();
           sql += "idcard = '"+number+"'";
       } else {
           // 1.2 若type为2则提示输入准考证号
           System.out.print("请输入准考证号：");
           number = scanner.next();
           sql += "examcard = '"+number+"'";
       }
       // 3. 执行查询
       Student student = getStudent(sql);
       // 4. 若存在查询结果，则封装为student对象
       return student;
   }
   /**
    * 根据传入的sql返回student对象
    * @param sql
    * @return
    */
   private Student getStudent(String sql) {
       Connection connection = null;
       Statement statement = null;
       ResultSet resultSet = null;
       Student student = null;
       try {
           connection = JDBCTools.getConnection();
           statement = connection.createStatement();
           resultSet = statement.executeQuery(sql);
           if(resultSet.next()){
               student = new Student(resultSet.getInt("flowid"),
                                     resultSet.getInt("type"),
                                     resultSet.getString("idCard"),
                                     resultSet.getString("examCard"),
                                     resultSet.getString("studentName"),
                                     resultSet.getString("location"),
                                     resultSet.getInt("grade"));
           }
       } catch (IOException e) {
           e.printStackTrace();
       } catch (ClassNotFoundException e) {
           e.printStackTrace();
       } catch (SQLException e) {
           e.printStackTrace();
       } finally {
           JDBCTools.release(resultSet,statement,connection);
       }
       return student;
   }
   ```

   

3. 打印学生信息

   ```java
   /**
    * 打印学生信息：若学生存在则打印其具体信息，若不存在则打印查无此人
    * @param student
    */
   private void printStudent(Student student) {
       if (student != null) {
           System.out.println(student);
       } else{
           System.out.println("查无此人");
       }
   }
   ```

   

4. 测试：

   ```java
   public static void main(String[] args) {
       JDBCTest jdbcTest = new JDBCTest();
       // 1. 得到查询的类型
       int searchType = jdbcTest.getSearchTypeFromCosole();
       // 2. 具体查询学生信息
       Student student = jdbcTest.searchStudent(searchType);
       // 3. 打印学生信息
       jdbcTest.printStudent(student);
   }
   ```




### 需求3

![删除练习](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/JDBC/06%E5%88%A0%E9%99%A4%E7%BB%83%E4%B9%A0.png)