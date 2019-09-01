---
2019-08-01 11:24:20

---



@Caching组合注解

```java
@Override
@Caching(
    cacheable = {
        @Cacheable(value = "emp",key = "#lastName")
    },
    put = {
        @CachePut(value = "emp",key = "#result.id"),
        @CachePut(value = "emp",key = "#result.email")
    }
)
public Employee getEmpByLastName(String lastName) {
    return employeeMapper.getEmpByLastName("Tim");
}
```

定义复杂的缓存规则，如把lastName作为key的同时，把其他字段也作为key，如果其他方法以这个key作为条件的时候，也会用到缓存。



@CacheConfig公共配置注解

这个注解打在**类**上，用于简化代码，比如都是用emp作为缓存组件【Cache】的，每个方法都声明，太过于麻烦，可以**用这个注解来指定公共配置。**

- cacheName
- keyGenerator
- cacheManager
- cacheResolver

指定公共配置。

