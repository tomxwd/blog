---
title: 04_SpringMvc_RequestMapping_请求方式
date: 2019-07-24 16:30:00
tags: 
 - Spring
 - SpringMvc
categories:
 - SpringMvc
---

# 04_SpringMvc_RequestMapping_请求方式

![映射请求参数请求方法或请求头](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/SpringMvc/04%E6%98%A0%E5%B0%84%E8%AF%B7%E6%B1%82%E5%8F%82%E6%95%B0%E8%AF%B7%E6%B1%82%E6%96%B9%E6%B3%95%E6%88%96%E8%AF%B7%E6%B1%82%E5%A4%B4.png)





![映射请求参数请求方法或请求头2](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/SpringMvc/04%E6%98%A0%E5%B0%84%E8%AF%B7%E6%B1%82%E5%8F%82%E6%95%B0%E8%AF%B7%E6%B1%82%E6%96%B9%E6%B3%95%E6%88%96%E8%AF%B7%E6%B1%82%E5%A4%B42.png)



RequestMethodController:

```java
@Controller
public class RequestMethodController {

    public static final String SUCCESS = "success";

    /**
     * 常用：使用method属性来指定请求方式
     * @return
     */
    @RequestMapping(value = "/method",method = RequestMethod.POST)
    public String testMethod(){
        System.out.println("RequestMethodController.testMethod");
        return SUCCESS;
    }


}
```



index.jsp:

```jsp
<form action="method" method="post">
    <input type="submit" value="submit">
</form>
```

