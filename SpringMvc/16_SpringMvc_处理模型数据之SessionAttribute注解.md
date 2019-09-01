---
title: 16_SpringMvc_处理模型数据之SessionAttribute注解
date: 2019-07-25 11:05:49
tags: 
 - Spring
 - SpringMvc
categories:
 - SpringMvc
---

# 16_SpringMvc_处理模型数据之SessionAttribute注解

## @SessionAttributes

- **若希望在多个请求之间共用某个模型属性数据**，则可以在控制器类上面标注一个@SessionAttributes注解，SpringMvc将会在模型中对应的属性暂存到HttpSession中。
- @SessionAttributes除了可以通过属性名指定需要放到会话中的属性外，还可以通过模型属性的对象类型指定那些模型属性需要放到会话中：
  - @SessionAttributes(type-User.class)会将隐含模型中所有类型为User.class的属性添加到会话中。
  - @SessionAttributes(value={"user1","user2"})
  - @SessionAttributes(types={User.class,Dept.class})
  - @SessionAttributes(value={"user1","user2"},types={Dept.class})

```java
/**
 * SessionAttributes两种用法 一种是key名，一种是类型，符合就会放在session域中
 * 注意：该注解要放在类上
 */
@SessionAttributes(value = {"user"},types = {String.class})
@Controller
public class RequestMethodController {

    public static final String SUCCESS = "success";

    @RequestMapping("/testSession")
    public String testSession(Map<String,Object> map){
        User user = new User();
        user.setUsername("root");
        user.setAge(11);
        user.setEmail("123@qc.com");
        user.setPassword("123");
        map.put("user",user);
        map.put("school","学");
        return SUCCESS;
    }
}
```

**注意：该注解要放在类上**