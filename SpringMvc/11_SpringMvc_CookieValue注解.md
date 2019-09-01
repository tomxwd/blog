---
title: 11_SpringMvc_CookieValue注解
date: 2019-07-25 09:58:16
tags: 
 - Spring
 - SpringMvc
categories:
 - SpringMvc
---

# 11_SpringMvc_CookieValue注解

## 使用@CookieValue绑定请求中的Cookie值

@CookieValue可以让处理方法入参绑定某个Cookie值

```java
/**
     * 了解：
     * @CookieValue：映射一个Cookie值
     * 属性同@RequestParam
     * @param sessionId
     * @return
*/
@RequestMapping("/cookie")
public String cookie(@CookieValue("JSESSIONID") String sessionId){
    System.out.println("RequestMethodController.cookie sessionId: "+sessionId);
    return SUCCESS;
}
```

