---
2019-08-01 15:32:07
---







测试缓存：

原理：

- CacheManager创建Cache组件，组件来实际的给缓存中存取数据

- 引入redis的starter，容器中保存的是RedisCacheManager；

- RedisCacheManager帮我们创建RedisCache作为缓存组件

- RedisCache通过操作redis来操作数据

- 启动项目，查询员工信息，再到redis中查看，发现默认还是使用序列化保存，如何保存为json形式

  - 引入了redis的starter，CacheManager变为RedisCacheManager
  - 默认创建的RedisCacheManager操作redis的时候使用的是RedisTemplate<Object,Object>
  - RedisTemplate<Object,Object>是RedisAutoConfiguration帮我们创建的，配置中默认使用的是JdkSerializationRedisSerializer【JDK序列化机制】

- 自定义CacheManager

  ```java
  /**
       * CacheManagerCustomizers可以来定制缓存的一些规则
       * @param redisTemplate
       * @return
  */
  @Bean
  public RedisCacheManager employeeCacheManager(RedisTemplate<Object, Employee> redisTemplate){
      RedisCacheManager redisCacheManager = new RedisCacheManager(redisTemplate);
      redisCacheManager.setUsePrefix(true);
      return redisCacheManager;
  }
  ```

- 启动项目，查看员工号为1的员工信息，然后查看redis，发现有一个key为emp:1的键，里面保存的就是我们查询的用户信息。这个key多了一个前缀，是因为在自定义CacheManager的时候开启了前缀配置`  redisCacheManager.setUsePrefix(true);`因此存入的时候会有CacheName作为key的前缀，可以用于区分；

- 这时候有个问题，我们定制的redisTemplate使用的是Employee作为value值，Department怎么办。

- 写好关于Department的cotroller、service、mapper后进行测试，发现第一次访问可以，第二次访问报错，报无法读取json数据的错，原因是映射错了对象，映射了Employee对象，所以报错。

- 关于上面Department数据可以存进去，但是取不出来，因为CacheManager默认是使用RedisTemplate<Object,Employee>操作Redis，相当于他只能对Employee进行反序列化操作。

- 理所当然的想法是再写多一个RedisTemplate<Object,Department>，但是不现实，如果有越来越多的对象需要缓存，就要不断加RedisTemplate。

- 先实现一个新的RedisTemplate<Object,Department>并实现一个新的RedisCacheManager；

  ```java
  @Bean
  public RedisTemplate<Object, Department> deptRedisTemplate(RedisConnectionFactory redisConnectionFactory) {
      RedisTemplate<Object,Department> template = new RedisTemplate<Object, Department>();
      template.setConnectionFactory(redisConnectionFactory);
      Jackson2JsonRedisSerializer<Department> serializer = new Jackson2JsonRedisSerializer<Department>(Department.class);
      template.setDefaultSerializer(serializer);
      return template;
  }
  
  @Bean
  public RedisCacheManager deptCacheManager(RedisTemplate<Object, Department> redisTemplate){
      RedisCacheManager redisCacheManager = new RedisCacheManager(redisTemplate);
      redisCacheManager.setUsePrefix(true);
      return redisCacheManager;
  }
  ```

  然后对DepartmentServiceImpl都指定cacheManager为“deptCacheManager",EmployeeServiceImpl指定为"employeeCacheManager";

- 启动项目报错，原因是在多个CacheManager的时候，需要指定一个主要的CacheManager，给其中一个CacheManager加上`@Primary`注解，重新启动项目；
- 再次访问，发现缓存生效了。
- 在开发中，肯定不是这样做的，直接弄一个RedisCacheManager以及RedisTemplate<Object,Object>处理所有的情况即可。



在开发中，我们经常需要在方法中调用缓存，怎么做？

引入对应的RedisCacheManager

```java
@Autowired
@Qualifier("deptCacheManager")
private RedisCacheManager deptCacheManager;
```

在代码中使用：

```java
@Override
public Department getDeptById(Integer id) {
    System.out.println("查询部门号为"+id);
    Department dept = departmentMapper.getDeptById(id);
    // 获取缓存
    Cache cache = deptCacheManager.getCache("dept");
    cache.put("dept:"+id,dept);
    return dept;
}
```

