---
title: 03_SpringMvc_RequestMapping_修饰类
date: 2019-07-24 16:18:40
tags: 
 - Spring
 - SpringMvc
categories:
 - SpringMvc
---

# 03_SpringMvc_RequestMapping_修饰类

## 使用@RequestMapping映射请求

- SpringMvc使用**@RequestMapping**注解为控制器指定可以处理那些URL请求
- 在控制器的**类定义及方法定义处**都可以标注
  - @RequestMapping
    - **类定义处：**提供初步的请求映射信息。相对于WEB应用的根目录。
    - **方法处：**提供进一步的细分映射信息。相对于类定义处的URL。若类定义处未标注@RequestMapping，则方法处标记的URL相对于WEB应用的根目录。
- DispatcherServlet截获请求之后，就通过控制器上的@RequestMapping提供的映射信息确定请求所对应的处理方法。



## 使用@RequestMapping映射请求示例

![使用@RequestMapping映射请求示例](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/SpringMvc/03%E4%BD%BF%E7%94%A8%40RequestMapping%E6%98%A0%E5%B0%84%E8%AF%B7%E6%B1%82%E7%A4%BA%E4%BE%8B.png)

SpringMvcTest:

```java
@RequestMapping("/springmvc")
@Controller
public class SpringMvcTest {

    private final String SUCCESS = "success";

    /**
     * 1. @RequestMapping 除了修饰方法还可以修饰类
     * 2. 类定义处：提供初步的请求映射信息，相对于WEB应用的根目录
     *    方法处：提供进一步的细分映射信息，相对于类定义处的URL，若类定义处未标注@RequestMapping
     *    则方法处标记的URL相对于WEB应用的根目录
     * @return
     */
    @RequestMapping("/testRequestMapping")
    public String testRequestMapping(){
        System.out.println("SpringMvcTest.testRequestMapping");
        return SUCCESS;
    }

}
```

