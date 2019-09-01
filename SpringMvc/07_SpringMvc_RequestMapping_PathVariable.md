---
title: 07_SpringMvc_RequestMapping_PathVariable
date: 2019-07-24 17:02:12
tags: 
 - Spring
 - SpringMvc
categories:
 - SpringMvc
---

# 07_SpringMvc_RequestMapping_PathVariable

## @PathVariable映射URL绑定的占位符

- 带占位符的URL是Spring3.0新增的功能，该功能在SpringMvc向REST目标挺进发展过程中具有里程碑的意义。
- 通过@PathVariable可以将URL中占位符参数绑定到控制器处理方法的入参中：URL中的{xxx}占位符可以通过@PathVariable("xxx")绑定到操作方法的入参中。

```java
/**
* @PathVariable 可以来映射URL中的占位符到目标方法的参数中
* @param id
* @return
*/    
@RequestMapping("/pathVariable/{id}")
public String testPathVariable(@PathVariable(value = "id") Integer id){
    System.out.println("RequestMethodController.testPathVariable"+id);
    return SUCCESS;
}
```

