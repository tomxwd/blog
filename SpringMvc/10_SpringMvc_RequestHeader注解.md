---
title: 10_SpringMvc_RequestHeader注解
date: 2019-07-25 09:40:48
tags: 
 - Spring
 - SpringMvc
categories:
 - SpringMvc
---

# 10_SpringMvc_RequestHeader注解

## 使用@RequestHeader绑定请求报头的属性值

请求头包含了若干个属性，服务器可以根据这个获取客户端的信息，**通过@RequestHeader即可将请求头中的属性值绑定到处理方法的入参中。**

```java
/**
     * 了解：
     * 作用：映射请求头信息
     * 用法同@RequestParam
     * @param al
     * @return
*/
@RequestMapping("/requestHeader")
public String testRequestHeader(@RequestHeader(value = "Accept-Language")String al){
    System.out.println("RequestMethodController.testRequestHeader Accept-Language :"+al);
    return SUCCESS;
}
```

   


