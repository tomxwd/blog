---
title: 01_SpringMvc_概述
date: 2019-07-24 14:40:54
tags: 
 - Spring
 - SpringMvc
categories:
 - SpringMvc
---

# 01_SpringMvc_概述

## 内容概要

1. SpringMvc概述
2. SpringMvc的HelloWorld
3. 使用@RequestMapping映射请求
4. 映射请求参数&请求头
5. 处理模型数据
6. 视图和视图解析器
7. RESTfulCRUD
8. SpringMvc表单标签&处理静态资源
9. 数据转换&数据格式化&数据校验
10. 处理JSON：使用HttpMessageConverter
11. 国际化
12. 文件的上传
13. 使用拦截器
14. 异常处理
15. SpringMvc运行流程
16. 在Spring的环境下使用SpringMvc
17. SpringMvc对比Struts2

## SpringMvc概述

- Spring为展现层提供的基于MVC设计理念的优秀的Web框架，是目前最主流的MVC框架之一
- Spring3.0后全面超越Struts2，称为最优秀的MVC框架
- SpringMvc通过一套MVC注解，让POJO成为处理请求的控制器，而无需实现任何接口
- 支持REST风格的URL请求
- 采用了松散耦合可插拔的组件结构，比其他MVC框架更具扩展性和灵活性