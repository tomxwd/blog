---
title: 12_SpringMvc_使用POJO作为参数
date: 2019-07-25 10:02:55
tags: 
 - Spring
 - SpringMvc
categories:
 - SpringMvc
---

# 12_SpringMvc_使用POJO作为参数

## 使用POJO对象绑定请求参数值

SpringMvc会按照请求的参数名和POJO属性名进行自动匹配，自动为该对象填充属性值。

支持级联属性，如：dept.deptId、dept.address.tel等。

```java
@RequestMapping(value = "testPojo",method = RequestMethod.POST)
public String testPojo(User user){
    System.out.println("RequestMethodController.testPojo User:"+user);
    return SUCCESS;
}
```

