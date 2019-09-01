---
title: 06_SpringMvc_RequestMapping_Ant路径
date: 2019-07-24 16:56:51
tags: 
 - Spring
 - SpringMvc
categories:
 - SpringMvc
---

# 06_SpringMvc_RequestMapping_Ant路径

![使用@RequestMapping映射请求](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/SpringMvc/06%E4%BD%BF%E7%94%A8%40RequestMapping%E6%98%A0%E5%B0%84%E8%AF%B7%E6%B1%82%E7%A4%BA%E4%BE%8B.png)

```java
@RequestMapping("antPath/*/abc")
public String testAntPath(){
    System.out.println("RequestMethodController.testAntPath");
    return SUCCESS;
}
```

