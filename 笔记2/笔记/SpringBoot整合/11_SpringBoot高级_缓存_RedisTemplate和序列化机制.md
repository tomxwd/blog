---
2019-08-01 13:51:52

---










整合Redis进SpringBoot：

1. 引入redis的starter

   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-data-redis</artifactId>
   </dependency>
   ```

2. 配置redis

   ```properties
   # redis
   spring.redis.host=tomxwd.top
   # 这里如果没改，默认6379
   spring.redis.port=12388
   ```

3. 测试：

   Redis常见的五大数据类型：

   - String（字符串） stringRedisTemplate.opsForValue();

     ```java
     @Test
     public void testStringRedisTemplate(){
         // 给redis中保存了数据
         Integer result = stringRedisTemplate.opsForValue().append("msg", "newHello!");
         System.out.println("result = " + result);
         // 读出数据
         String msg = stringRedisTemplate.opsForValue().get("msg");
         System.out.println("msg = " + msg);
     }
     ```

   - List（列表）stringRedisTemplate.opsForList();

     ```java
     @Test
     public void testStringRedisTemplate(){
         stringRedisTemplate.opsForList().leftPush("mylist","1");
         stringRedisTemplate.opsForList().leftPush("mylist","2");
         String msg1 = stringRedisTemplate.opsForList().leftPop("mylist");
         String msg2 = stringRedisTemplate.opsForList().leftPop("mylist");
         System.out.println("msg1 = " + msg1);
         System.out.println("msg2 = " + msg2);
     }
     ```

   - Set （集合）stringRedisTemplate.opsForSet();

   - Hash （散列）stringRedisTemplate.opsForHash();

   - ZSet （有序集合）stringRedisTemplate.opsForZSet();

   - 测试存入一个对象：

     ```java
     @Test
     public void testRedisTemplate(){
         redisTemplate.opsForValue().set("emp-01",employeeMapper.getEmpById(1));
     }
     ```

     结果报错：

     ```
     DefaultSerializer requires a Serializable payload but received an object of type [top.tomxwd.cache.bean.Employee]
     ```

     Employee没有实现序列化，对Employee实现序列化接口，再运行测试。

     发现默认保存对象，使用JDK序列化机制，序列化后的数据保存到redis中。

     但是一般我们想要用json的形式保存在redis中。

4. 改为自己定义的序列化器（默认是JDK序列化）

   改变默认的序列化规则；

   ```java
   @Configuration
   public class MyRedisConfig {
   
       @Bean
       public RedisTemplate<Object, Employee> empRedisTemplate(RedisConnectionFactory redisConnectionFactory) {
           RedisTemplate<Object,Employee> template = new RedisTemplate<Object, Employee>();
           template.setConnectionFactory(redisConnectionFactory);
           Jackson2JsonRedisSerializer<Employee> serializer = new Jackson2JsonRedisSerializer<Employee>(Employee.class);
           template.setDefaultSerializer(serializer);
           return template;
       }
   
   }
   ```

   再运行：

   ```java
   @Test
   public void testRedisTemplate(){
       // 默认如果保存对象，使用JDK的序列化机制，序列化后的数据保存在redis中
       // redisTemplate.opsForValue().set("emp-01",employeeMapper.getEmpById(1));
       // 1. 将数据以json的方式保存
       // （1）自己将对象转为json
       // （2）redistTemplate默认的序列化规则（JDK），切换为自己的序列化规则
       empRedisTemplate.opsForValue().set("emp-01",employeeMapper.getEmpById(1));
   }
   ```

   再查看结果；