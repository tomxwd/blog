---
title: 24_SpringMvc_视图解析流程分析
date: 2019-07-25 15:08:47
tags: 
 - Spring
 - SpringMvc
categories:
 - SpringMvc
---

# 24_SpringMvc_视图解析流程分析

## SpringMvc如何解析视图

![SpringMvc如何解析视图1](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/SpringMvc/24SpringMvc%E5%A6%82%E4%BD%95%E8%A7%A3%E6%9E%90%E8%A7%86%E5%9B%BE1.png)

![SpringMvc如何解析视图2](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/SpringMvc/24SpringMvc%E5%A6%82%E4%BD%95%E8%A7%A3%E6%9E%90%E8%A7%86%E5%9B%BE2.png)

## 视图和视图解析器

- 请求处理方法执行完成后，最终返回一个ModelAndView对象。对于那些返回String，View或者ModelMap等类型的处理方法，**SpringMvc也会在内部将它们装配成一个ModelAndView对象**，它包含了逻辑名和模型对象的视图。
- SpringMvc借助**视图解析器（ViewResolver）**得到最终的视图对象（View），最终的视图可以是JSP，也可能是Excel、JFreeChart等各种表现格式的视图。
- 对于最终究竟采取何种视图对象模型数据进行渲染，处理器并不关心，处理器工作重点聚焦在生产模型数据的工作上，从而实现MVC的充分解耦。

### 视图

![视图](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/SpringMvc/24%E8%A7%86%E5%9B%BE.png)

#### 常用的视图实现类

![常用的视图实现类](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/SpringMvc/24%E5%B8%B8%E7%94%A8%E7%9A%84%E8%A7%86%E5%9B%BE%E5%AE%9E%E7%8E%B0%E7%B1%BB.png)

### 视图解析器

![视图解析器](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/SpringMvc/24%E8%A7%86%E5%9B%BE%E8%A7%A3%E6%9E%90%E5%99%A8.png)

#### 常用的视图解析器实现类

![常用的视图解析器实现类](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/SpringMvc/24%E5%B8%B8%E7%94%A8%E7%9A%84%E8%A7%86%E5%9B%BE%E8%A7%A3%E6%9E%90%E5%99%A8%E5%AE%9E%E7%8E%B0%E7%B1%BB.png)

##### InternalResourceViewResolver

![InternalResourceViewResolver](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/SpringMvc/24InternalResourceViewResolver.png)

调用一个方法返回String类型，View类型或其他类型，SpringMvc都会转为ModelAndView类型，通过视图解析器得到真正的物理视图View对象，最终调用View的render方法得到响应结果，常用的视图InternalResourceView，常用的视图解析器InternalResourceViewResolver，转发之后的结果。

