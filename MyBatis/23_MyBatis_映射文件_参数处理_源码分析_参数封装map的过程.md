---
title: 23_MyBatis_映射文件_参数处理_源码分析_参数封装map的过程
date: 2019-08-10 11:27:33
tags: 
 - Java
 - Mybatis
categories:
 - Mybatis
---

# 23_MyBatis\_映射文件\_参数处理\_源码分析_参数封装map的过程

结合源码，查看mybatis是如何对参数进行处理的

```java
(@Param("id") Integer id,@Param("lastName") String lastName);
```

- ParamNameReslover解析参数封装map；

- names：{0=id，1=lastName}，构造器的时候就完成了赋值。

  确定赋值流程：

  1. 获取每个标了Param注解的参数的值：id，lastName，赋值给name；

  2. 每次解析一个参数给map中保存信息：（key：参数索引，value：name值）

     name的值：

     - 标注了param的注解：注解指定的值。

     - 全局配置文件配了UseActualParamName，就按照参数名的值。（**需要jdk1.8及以上**）
     - 啥都没有，就是map.size()，也就是相当于索引0,1,2...
     - 
     - 



## ParamNameResolver(Configruation config, Method method)

```java
public ParamNameResolver(Configuration config, Method method) {
    final Class<?>[] paramTypes = method.getParameterTypes();
    final Annotation[][] paramAnnotations = method.getParameterAnnotations();
    final SortedMap<Integer, String> map = new TreeMap<Integer, String>();
    int paramCount = paramAnnotations.length;
    // get names from @Param annotations
    for (int paramIndex = 0; paramIndex < paramCount; paramIndex++) {
        if (isSpecialParameter(paramTypes[paramIndex])) {
            // skip special parameters
            continue;
        }
        String name = null;
        for (Annotation annotation : paramAnnotations[paramIndex]) {
            // 判断注解是不是Param
            if (annotation instanceof Param) {
                hasParamAnnotation = true;
                name = ((Param) annotation).value();
                break;
            }
        }
        if (name == null) {
            // @Param was not specified.
            if (config.isUseActualParamName()) {
                name = getActualParamName(method, paramIndex);
            }
            if (name == null) {
                // use the parameter index as the name ("0", "1", ...)
                // gcode issue #71
                name = String.valueOf(map.size());
            }
        }
        map.put(paramIndex, name);
    }
    names = Collections.unmodifiableSortedMap(map);
}
```





## ParamNameResolver.getNamedParams()

```java
/**
   * <p>
   * A single non-special parameter is returned without a name.<br />
   * Multiple parameters are named using the naming rule.<br />
   * In addition to the default names, this method also adds the generic names (param1, param2,
   * ...).
   * </p>
*/
public Object getNamedParams(Object[] args) {
    final int paramCount = names.size();
    // 如果参数为空，那么直接返回null；
    if (args == null || paramCount == 0) {
        return null;
        // 如果只有一个元素，且没有@Param注解，那么直接返回 args[0],单个参数直接返回
    } else if (!hasParamAnnotation && paramCount == 1) {
        return args[names.firstKey()];
        // 如果多个元素或者有@Param注解
    } else {
        final Map<String, Object> param = new ParamMap<Object>();
        int i = 0;
        // 遍历names集合，{0=id,1=lastName}
        for (Map.Entry<Integer, String> entry : names.entrySet()) {
            // names集合的value作为key；names集合的key又作为取值的参考args[0]
            // 就是param={id:1,lastName=Tom}
            // 假设有第三个参数，且没有@Param注解，那么就会变成param={id:1,lastName=Tom,2=args[2]}
            param.put(entry.getValue(), args[entry.getKey()]);
            // add generic param names (param1, param2, ...)
            // 这里GENERIC_NAME_PREFIX=param，就是除了上面的形式之外，额外的用param+index的形式再封装到map中，key=param+index，所以最后有两种形式可以取得
            final String genericParamName = GENERIC_NAME_PREFIX + String.valueOf(i + 1);
            // ensure not to overwrite parameter named with @Param
            if (!names.containsValue(genericParamName)) {
                param.put(genericParamName, args[entry.getKey()]);
            }
            i++;
        }
        return param;
    }
}
```



## 小结

参数多的时候mybatis会封装成map来处理，为了不混乱，我们用@Param注解来指定封装时候用的key，那么在使用的时候用#{key}就可以取出值。