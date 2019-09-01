---
title: 00_Apache-beanutis_注意事项
date: 2019-07-29 09:49:19
tags: 
 - Java
 - Apache-beanutis
categories:
 - Apache-beanutis
---

# 00_Apache-beanutis_注意事项

## 避免使用Apache Beanutils进行属性copy

![避免使用ApacheBeanutils进行属性copy](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/%E5%B7%A5%E5%85%B7%E7%B1%BB/Apache-beanutils/00%E9%81%BF%E5%85%8D%E4%BD%BF%E7%94%A8ApacheBeanutils%E8%BF%9B%E8%A1%8C%E5%B1%9E%E6%80%A7copy.png)

性能低下，Cglib BeanCopier效率最高。



## pubilc修饰的类才能生效

在进行测试的时候，由于处于方便，直接在下面建了一个class声明，结果进行属性copy的时候为null，把class用pubilc声明出去就可以了。



## 与Spring的copyProperties区别

关于Apache的copyProperties(final Object dest, final Object orig)，前面的是等待接收属性的，后面的是提供属性的。而Spring的copyProperties虽然同名，但是顺序整好相反。



## 属性复制的时候Null会被初始化

如果被复制的值为null，复制到目标对象的时候，会被初始化，**比如：**

- **Integer，BigDecimal原本为null，会被初始化为0，如果是在搜索条件情况下，会造成影响。**
- **Date会报错**
- Boolean初始化为false

解决办法(使用**ConvertUtils.register**)：

```java
ConvertUtils.register(new DateConverter(null), java.util.Date.class);  
ConvertUtils.register(new BigDecimalConverter(null), java.math.BigDecimal.class);  
ConvertUtils.register(new IntegerConverter(null), java.lang.Integer.class);  
ConvertUtils.register(new BooleanConverter(null), java.lang.Boolean.class);
```

这样，这些类就不会被初始化了。

经过使用发现：

- 使用1.9.3并不会初始化，但ConvertUtils也无法生效。
- 使用1.8.3就会出现上述问题，但ConvertUtils可以生效。