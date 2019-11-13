---
title: 04_tomcat_进阶改造
date: 2019-11-09 18:12:28
tags: 
 - tomcat
categories:
 - tomcat
---

# 04_tomcat_进阶改造

## 编写BaseServlet

一个Servlet只有一套doGet和doPost方法，那么如何让一个Servlet处理多个功能：

- 第一种就是直接在请求参数里加一个method作为判断，根据判断结果做出对应操作；

  - 看上去没问题，但是实际存在很大问题：
    - get请求，表单的数据会被带上，但是请求地址的数据会被表单元素覆盖，可以用隐藏的input框来解决。
    - 每个Servlet会存在大量的method判断，存在多个方法；

- 第二种，基于第一种的形式，利用反射解决if判断

  - ```java
    // 获取该类声明的方法，（方法名，参数...）
    Method m = this.getClass().getDeclareMethod(method,HttpServletRequest.class,HttpServletResponse.class);
    // 把方法权限设置大
    m.setAccessible(true);
    // invoke(对象，参数)
    m.invoke(this,req,resp);
    ```

- 第三种，基于第二种再次改造，抽取出BaseServlet，用于被所有Servlet继承

  ```java
  public class BaseServlet extends HttpServlet {
  
      @Override
      protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
          doPost(req,resp);
      }
  
      @Override
      protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
          String methodName = req.getParameter("method");
          try {
              // 获取该类声明的方法，（方法名，参数...）
              Method method = this.getClass().getDeclaredMethod(methodName, HttpServletRequest.class, HttpServletResponse.class);
              // 把方法权限设置大
              method.setAccessible(true);
              // invoke(对象，参数)
              method.invoke(this,req,resp);
          } catch (Exception e) {
              e.printStackTrace();
          }
      }
      
  }
  ```

  - 这样，子类只要继承BaseServlet，然后编写相应的方法即可，页面需要调用的之后只要指定方法名就可以。



## 参数封装优化

继续抽取一个自动化工具，根据前台传入的参数信息，自动封装数据到对象中，可以用反射来做。

用到apache工具类：

```xml
<!-- beanutils -->
<dependency>
    <groupId>commons-beanutils</groupId>
    <artifactId>commons-beanutils</artifactId>
    <version>1.8.0</version>
</dependency>
```

WebUtils类：

不用beanutils做：

```java
public class WebUtils {

    /**
     * 传入request对象，将request中的数据封装给对象
     * @param req
     * @param t
     * @param <T>
     * @return
     */
    public static <T> T param2bean(HttpServletRequest req, T t) {
        // 封装对象并返回
        // 1. 获取所有属性
        Field[] fields = t.getClass().getFields();
        // 2. 每个属性都有name值，属性名
        for (Field field : fields) {
            // 获取属性名
            String name = field.getName();
            // 在request中根据name取出值
            String value = req.getParameter("name");
            // 封装对象（用setAttrName方法）


        }
        return null;
    }
}
```

但是需要做很麻烦的操作，因为request获取的都是String类型，需要做类型判断然后强转；

而beanutils已经做好了这一步，而且不对应类型就不做操作，使用方便；

使用beanutils做：

```java
public class WebUtils {

    /**
     * 传入request对象，将request中的数据封装给对象
     *
     * @param req
     * @param t
     * @param <T>
     * @return
     */
    public static <T> T param2bean(HttpServletRequest req, T t) {
        Field[] fields = t.getClass().getFields();
        for (Field field : fields) {
            String name = field.getName();
            String value = req.getParameter("name");
            try {
                BeanUtils.setProperty(t, name, value);
            } catch (IllegalAccessException e) {
                e.printStackTrace();
            } catch (InvocationTargetException e) {
                e.printStackTrace();
            }
        }
        return t;
    }
}
```

但是还是觉得用了一些底层的反射代码，进行改造：

```java
public class WebUtils {

    /**
     * 传入request对象，将request中的数据封装给对象
     *
     * @param req
     * @param t
     * @param <T>
     * @return
     */
    public static <T> T param2bean(HttpServletRequest req, T t) {
        // populate将map中的键值对直接映射到javabean中去
        Map<String, String[]> parameterMap = req.getParameterMap();
        try {
            BeanUtils.populate(t,parameterMap);
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (InvocationTargetException e) {
            e.printStackTrace();
        }
        return t;
    }
    
}
```

