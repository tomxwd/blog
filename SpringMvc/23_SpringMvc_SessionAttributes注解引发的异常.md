---
title: 23_SpringMvc_SessionAttributes注解引发的异常
date: 2019-07-25 14:53:32
tags: 
 - Spring
 - SpringMvc
categories:
 - SpringMvc
---

# 23_SpringMvc_SessionAttributes注解引发的异常

要解决这个异常很简单

- 要么session中已经有这个值。
- 要么就改入参处的@ModelAttribute的value值，让他跟@SessionAttributes的value值对应不上即可。
- 要是确实就是要同样的value值，那就用@ModelAttribute修饰的方法，并且，这个方法内要对这个key进行赋值，例如入参需要的key是user，而这个方法也要map.put("user",value)，否则还是会去找@SessionAttributes。