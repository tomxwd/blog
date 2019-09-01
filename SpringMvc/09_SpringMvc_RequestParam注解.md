---
title: 09_SpringMvc_RequestParam注解
date: 2019-07-25 09:29:12
tags: 
 - Spring
 - SpringMvc
categories:
 - SpringMvc
---

# 09_SpringMvc_RequestParam注解

## 请求处理方法签名

- SpringMvc通过分析处理方法的签名，将HTTP请求信息绑定到处理方法的相应入参中。
- SpringMvc对控制器处理方法签名的限制是很宽松的，几乎可以按照喜欢的任何方式对方法进行签名。
- 必要时可以对方法以及方法入参标注相应的注解
  - @PathVariable
  - @RequestParam
  - @RequestHeader
  - 等等
- SpringMvc框架会将HTTP请求的信息绑定到相应的方法入参中，并根据方法的返回值类型做出相应的后续处理。



## 使用@RequestParam绑定请求参数值

在处理方法入参处使用@RequestParam可以把请求参数传递给请求方法

- value：参数名。
- required：是否必须，默认是true，表示请求参数中必须包含对应的参数，若不存在，将抛出异常。
- defaultValue：默认值，配合required为false的时候。

```java
/**
      * @RestParam来映射请求参数
      * value与name等价 值即为请求参数的参数名
      * required 该参数是否必须，默认为true
      * defaultValue 请求值的默认参数
      *
      * @param username
      * @param age
      * @return
*/
@RequestMapping(value = "/testParam")
public String testParam(@RequestParam(value = "username") String username,@RequestParam(value = "age",required = false,defaultValue = "0") int age){
    System.out.println("RequestMethodController.testParam username: "+username+" age: "+age);
    return SUCCESS;
}
```

   