---
2019-07-31 14:03:53
---



1. 创建springboot项目

   - web模块
   - mysql
   - mybatis
   - spring cache abstraction
   - Lombok
   - SpringBoot版本选1.5.21

2. 导入数据库文件

   - department表

     ```mysql
     CREATE TABLE `department` (
       `id` int(11) NOT NULL AUTO_INCREMENT,
       `department_name` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci DEFAULT NULL,
       PRIMARY KEY (`id`)
     ) ENGINE=InnoDB AUTO_INCREMENT=10 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci ROW_FORMAT=DYNAMIC;
     ```

   - employee表

     ```mysql
     CREATE TABLE `employee` (
       `id` int(11) NOT NULL AUTO_INCREMENT,
       `lastName` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci DEFAULT NULL,
       `email` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci DEFAULT NULL,
       `gender` int(11) DEFAULT NULL,
       `d_id` int(11) DEFAULT NULL,
       PRIMARY KEY (`id`)
     ) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci ROW_FORMAT=DYNAMIC;
     ```

3. 编写javabean

   - Department

     ```java
     @Data
     @AllArgsConstructor
     @RequiredArgsConstructor
     public class Department {
     
         private Integer id;
         private String departmentName;
     
     }
     ```

   - Employee

     ```java
     @Data
     @RequiredArgsConstructor
     @AllArgsConstructor
     public class Employee {
     
         private Integer id;
         private String lastName;
         private String email;
         private Integer gender;
         private Integer dId;
     
     }
     ```

4. 整合mybatis操作数据库

   - 配置数据源信息

     ```properties
     spring.datasource.url=jdbc:mysql://tomxwd.top:12389/spring_cache?useSSL=false
     spring.datasource.driver-class-name=com.mysql.jdbc.Driver
     spring.datasource.username=root
     spring.datasource.password=root
     ```

   - 配置驼峰命名规则

     ```properties
     # 开启驼峰命名匹配规则
     mybatis.configuration.map-underscore-to-camel-case=true
     ```

   - 使用注解版本的mybatis即可

     - @MapperScan指定需要扫描的mapper接口所在的包位置

       ```java
       @SpringBootApplication
       @MapperScan(basePackages = {"top.tomxwd.cache.mapper"})
       public class Springboot01CacheApplication {
       
           public static void main(String[] args) {
               SpringApplication.run(Springboot01CacheApplication.class, args);
           }
       
       }
       ```

     - 在各个XxxMapper上打@Mapper表示这是一个Mapper类，并写增删改查方法

       - EmployeeMapper

         ```java
         @Mapper
         public interface EmployeeMapper {
         
             @Select("SELECT id,lastName,email,gender,d_id dId FROM employee WHERE id=#{id}")
             Employee getEmpById(Integer id);
         
             @Update("UPDATE employee SET lastName = #{lastName},email=#{email},gender=#{gender},d_id=#{dId} WHERE id=#{id}")
             Integer updateEmp(Employee employee);
         
             @Delete("DELETE FROM employee WHERE id = #{id}")
             Integer deleteEmpById(Integer id);
         
             @Insert("INSERT INTO employee(lastName,email,gender,d_id) VALUES(#{lastName},#{email},#{gender},#{dId})")
             Integer insertEmp(Employee employee);
         
         }
         ```

     - 测试一下功能

       ```java
       @RunWith(SpringRunner.class)
       @SpringBootTest
       public class Springboot01CacheApplicationTests {
       
           @Autowired
           private EmployeeMapper employeeMapper;
       
           @Test
           public void contextLoads() {
       
               Employee empById = employeeMapper.getEmpById(1);
               System.out.println(empById);
       
           }
       
       }
       ```

       结果：

       ```
       Employee(id=1, lastName=TOM, email=TOM@qq.com, gender=0, dId=1)
       ```

5. 加上service层和controller层

   - service

     ```java
     public interface EmployeeService {
         Employee getEmp(Integer id);
     }
     ```

   - serviceImpl

     ```java
     @Service
     public class EmployeeServiceImpl implements EmployeeService {
     
         @Autowired
         private EmployeeMapper employeeMapper;
     
         @Override
         public Employee getEmp(Integer id) {
             System.out.printf("查询%d号员工",id);
             Employee emp = employeeMapper.getEmpById(id);
             return emp;
         }
     }
     ```

   - controller

     ```java
     @RestController
     public class EmployeeController {
     
         @Autowired
         private EmployeeService employeeService;
     
         @GetMapping("/emp/{id}")
         public Employee getEmployee(@PathVariable("id") Integer id){
             Employee emp = employeeService.getEmp(id);
             return emp;
         }
     }
     ```

6. 运行项目，访问地址`localhost:8080/emp/1`返回json则已配置完成。