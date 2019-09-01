---
title: 23_Shiro_会话管理
date: 2019-08-30 15:42:36
tags: 
 - Java
 - Shiro
categories:
 - Shiro
---

# 23_Shiro_会话管理

## 概述

Shiro提供了完整的企业级会话管理功能，**不依赖于底层容器**（如web容器tomcat），**不管JavaSE还是JavaEE环境都可以使用**，提供了会话管理、会话事件监听、会话存储/持久化、容器无关的集群、失效/过期支持、对Web的透明支持、SSO单点登录的支持等特性；



## 会话相关的API

- **Subject.getSession()**：即可获取会话；其等价于Subject.getSession(true)，即如果当前没有创建Session对象会创建一个；Subject.getSession(false)，如果当前没有创建Session则返回null；
- **session.getId()**：获取当前会话的唯一标识；
- **session.getHost()**：获取当前Subject的主机地址；
- **session.getTimeout()&session.setTimeout(毫秒)**：获取/设置当前Session的过期时间；
- **session.getStartTimestamp()&session.getLastAccessTime()**：获取会话的启动时间以及最后访问时间；如果是JavaSE应用需要自己定期调用session.touch()去更新最后的访问时间，如果是Web应用，每次进行ShiroFilter都会自动调用session.touch()来更新最后访问时间；
- **session.touch()&session.stop()**：更新会话最后访问时间以及销毁会话；当Subject.logout()的时候会自动调用stop方法来销毁会话。如果在web中，调用HttpSession.invalidate()也会自动调用Shiro的session.stop()方法进行销毁Shiro的会话；
- **session.setAttribute(key,val)&session.getAttribute(key)&session.removeAttribute(key)**：设置/获取/删除会话属性；在整个会话范围内都可以对这些属性进行操作；



## 会话监听器

会话监听器用于监听会话创建、过期以及停止事件；

SessionListener(Interface)

- onStart(Session)：void
- onStop(Session)：void
- onExpiration(Session)：void



## 利用ShiroSession的好处

利用Shiro的Session可以做到，在service层拿到session，降低耦合：

controller层：

```java
@RequestMapping("/testShiroAnnotation")
public String testShiroAnnotation(HttpSession session){
    session.setAttribute("key","value12345");
    service.testMethod();
    return "redirect:/list.jsp";
}
```

service层：

```java
@RequiresRoles(value = {"admin"})
public void testMethod(){
    System.out.println("testMethod,time :"+new Date());
    Session session = SecurityUtils.getSubject().getSession();
    Object value = session.getAttribute("key");
    System.out.println("key = " + value);
}
```



