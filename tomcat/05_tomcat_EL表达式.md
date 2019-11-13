---
title: 05_tomcat_EL表达式
date: 2019-11-09 19:43:47
tags: 
 - tomcat
categories:
 - tomcat
---

# 05_tomcat_EL表达式

EL全名为Expression Language，它可以在JSP页面上直接使用；

格式：${表达式内容}

例如：

```java
value="${requestScope.emp.empName}"
```

相当于

```html
<% Employee emp = (Employee)request.getAttribute("emp"); %>
    员工姓名<input type="text" name="empName" value="<%=emp.getEmpName()%>" >
```



## EL表达式的功能

1. **在页面显示域中的属性值**
   - ${输入属性名}
2. **要获取对象中的某个属性直接使用.属性名的方式（还是根据getter、setter取出来的）**
3. **el表达式如果获取域中的属性，直接写属性名，会从4个域中从小到大找，找到就停止（可以用隐含对象取指定的域）**
4. **el隐含对象（11个）**
   - **四个域对象：**
     - pageScope：page域中的数据，是一个map，可以用.属性名的方式获取
     - requestScope：request域中的数据，也是一个map
     - sessionScope：session域中的数据，也是一个map
     - applicationScope：application域中的数据，也是一个map
   - **一个非map对象：**
     - pageContext：就是JSP页面上的pageContext，可以取出jsp页面其他的隐含对象（九大隐含对象），但是是用.的形式取。
   - **和HTTP协议有关：**
     - param：map类型，对应一个请求参数，request.getParamer("username")
     - paramValues：map类型，对应一组请求参数，数组（${paramValues.ah[0]},${paramValues.ah[1]}）等
     - header：map类型，请求头，request.getHeader("User-Agent")
     - headerValues：map类型，请求头返回字符数组
     - cookie：map类型，获取某个cookie对象，取出cookie的值
   - **获取初始化参数：**
     - initParam：map类型，获取应用初始化参数，获取web的初始化参数，在web.xml中初始化参数
5. **el表达式如何获取stu-x这些特殊命名的属性（因为会产生歧义，如减号等）**
   - 使用[‘stu-x’]取出
     - `${requestScope['stu-x']}`
6. **el表达式运算**
   - 算术运算：${5+3}
   - 关系运算：${5>3}
   - 逻辑运算：${true&&false}
   - empty运算：
     - ${empty requestScope.emp}
       - null
         - 变量的值是null
         - 域对象中不存在这个变量
       - 空集
       - 空数组
       - 空字符串
       - 空字符
   - 三目条件运算：${16<5?'a':'big'}
7. **获取项目虚拟路径**
   - 获取request对象：${pageContext.request}
   - 获取contextPath：${pageContext.request.contextPath}