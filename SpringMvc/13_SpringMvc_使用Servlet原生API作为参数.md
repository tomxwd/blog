---
title: 13_SpringMvc_使用Servlet原生API作为参数
date: 2019-07-25 10:17:44
tags: 
 - Spring
 - SpringMvc
categories:
 - SpringMvc
---

# 13_SpringMvc_使用Servlet原生API作为参数

## 使用ServletAPI作为入参

MVC中的Handler方法可以接受以下ServletAPI类型的参数：

1. HttpServletRequest
2. HttpServletResponse
3. HttpSession
4. java.security.Principal
5. Locale
6. InputStream
7. OutputStream
8. Reader
9. Writer

```java
/**
     * 可以使用Servlet的原生API作为目标方法的参数
     * 具体支持以下类型
     * HttpServletRequest
     * HttpServletResponse
     * HttpSession
     * java.security.Principal
     * Locale
     * InputStream
     * OutputStream
     * Reader
     * Writer
*/
@RequestMapping(value = "stream")
public void testStream(Writer out, Locale local) throws IOException {
    System.out.println(local);
    out.write("hello,springmvc");
}


@RequestMapping(value = "servlet")
public String testServletApi(HttpServletRequest request,
                             HttpServletResponse response){
    System.out.println(request+" "+response);
    return SUCCESS;
}
```



