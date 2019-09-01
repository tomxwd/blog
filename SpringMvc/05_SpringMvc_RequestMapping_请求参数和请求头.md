---
title: 05_SpringMvc_RequestMapping_请求参数和请求头
date: 2019-07-24 16:37:05
tags: 
 - Spring
 - SpringMvc
categories:
 - SpringMvc
---

# 05_SpringMvc_RequestMapping_请求参数和请求头

![映射请求参数请求方法或请求头3](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/SpringMvc/05%E6%98%A0%E5%B0%84%E8%AF%B7%E6%B1%82%E5%8F%82%E6%95%B0%E8%AF%B7%E6%B1%82%E6%96%B9%E6%B3%95%E6%88%96%E8%AF%B7%E6%B1%82%E5%A4%B43.png)





```java
/**
* 了解：可以使用params和headers来更加精确映射请求
* params和headers支持简单的表达式。
* @return
*/
@RequestMapping(value = "/paramsAndHeaders",
                params = {"username","age!=10"},
                headers = {"Accept-Language=zh-CN,zh;q=0.9"})
public String testParamsAndHeaders(){
    System.out.println("RequestMethodController.testParamsAndHeaders");
    return SUCCESS;
}
```

