---
title: 15_SpringMvc_处理模型数据之Map
date: 2019-07-25 11:03:19
tags: 
 - Spring
 - SpringMvc
categories:
 - SpringMvc
---

# 15_SpringMvc_处理模型数据之Map

## Map及Model

![Map以及Model](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/SpringMvc/15Map%E4%BB%A5%E5%8F%8AModel.png)

```java
/**
 * 目标方法可以添加Map类型（实际上也可以是Model类型或ModelMap类型）的参数
 * @param map
 * @return
 */
@RequestMapping("/map")
public String testMap(Map<String,Object> map){
    map.put("names", Arrays.asList("Tom","Jerry","Mike"));
    return SUCCESS;
}
```