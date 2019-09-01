---
title: 14_SpringMvc_处理数据模型之ModelAndView
date: 2019-07-25 10:28:29
tags: 
 - Spring
 - SpringMvc
categories:
 - SpringMvc
---

# 14_SpringMvc_处理数据模型之ModelAndView

## 处理模型数据

SpringMvc提供了以下几种途径输出模型数据：

- ModelAndView：处理方法返回值类型为ModelAndView的时候，方法体即可通过该对象添加模型参数。
- Map、Model、ModelMap：入参为org.springframework.ui.Model、org.springframework.ui.ModelMap或java.util.Map时，处理方法返回时，Map中的数据会自动添加到模型中。
- @SessionAttributes：将模型中的某个属性暂存到HttpSession中，以便多个请求之间可以共享到这个属性。
- @ModelAttribute：方法入参标注该注解后，入参的对象就会放到数据模型中。



## ModelAndView

控制器处理方法的返回值如果是ModelAndView，则其既包含视图信息，也包含模型数据信息。

- 添加模型数据：
  - ModelAndView.addObject(String attributeName,Object attributeValue)；
  - ModelAndView.addAllObject(Map<String,?> modelMap)；
- 设置视图
  - void setView(View view)
  - void setViewName(String viewName)

```java
/**
     * 目标方法的返回值可以是ModelAndView类型
     * 其中可以包含视图和模型信息
     * SpringMvc会把ModelAndView的model中的数据放入到request域对象中。
     * @return
*/
@RequestMapping("testMv")
public ModelAndView testMv(){
    String viewName = SUCCESS;
    ModelAndView mv = new ModelAndView(viewName);
    // 添加模型数据到ModelAndView中
    mv.addObject("time",new Date());
    return mv;
}
```




