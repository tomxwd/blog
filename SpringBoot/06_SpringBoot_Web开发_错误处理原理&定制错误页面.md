---
title: 06_SpringBoot_Web开发_错误处理原理&定制错误页面
date: 2019-07-09 20:15:38
tags: 
 - Java
 - SpringBoot
categories:
 - Spring
 - SpringBoot初学
---

# 06_SpringBoot_Web开发_错误处理原理&定制错误页面

> 定制错误页面以及原理讲解



## SpringBoot_Web开发的错误页面默认效果

1. 浏览器：

   返回一个默认的错误页面，显示时间，错误码及类型。

   ![默认错误页面](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/SpringBoot/06%E9%BB%98%E8%AE%A4%E9%94%99%E8%AF%AF%E9%A1%B5%E9%9D%A2.png)

   浏览器发送的请求头：

   ![浏览器错误默认请求头信息](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/SpringBoot/06%E6%B5%8F%E8%A7%88%E5%99%A8%E9%94%99%E8%AF%AF%E9%BB%98%E8%AE%A4%E8%AF%B7%E6%B1%82%E5%A4%B4%E4%BF%A1%E6%81%AF.png)

2. 其他客户端，通过postman测试，发现默认返回的是json数据

   ![其他客户端返回json数据](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/SpringBoot/06%E5%85%B6%E4%BB%96%E5%AE%A2%E6%88%B7%E7%AB%AF%E8%BF%94%E5%9B%9Ejson%E6%95%B0%E6%8D%AE.png)

   其他客户端发送请求的请求头：

   ![其他客户端发送的请求头](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/SpringBoot/06%E5%85%B6%E4%BB%96%E5%AE%A2%E6%88%B7%E7%AB%AF%E5%8F%91%E9%80%81%E7%9A%84%E8%AF%B7%E6%B1%82%E5%A4%B4.png)

### 返回错误页面原理

- 点开Maven依赖找到springboot自动配置依赖

  ![SpringBoot包结构](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/SpringBoot/06SpringBoot%E5%8C%85%E7%BB%93%E6%9E%84.png)

- 找到web模块下的ErrorMvcAutoConfiguration

  给容器中添加了以下组件

  - DefaultErrorAttributes：

    ```java
    // 帮我们在页面共享信息
    public Map<String, Object> getErrorAttributes(WebRequest webRequest, boolean includeStackTrace) {
        Map<String, Object> errorAttributes = new LinkedHashMap();
        errorAttributes.put("timestamp", new Date());
        this.addStatus(errorAttributes, webRequest);
        this.addErrorDetails(errorAttributes, webRequest, includeStackTrace);
        this.addPath(errorAttributes, webRequest);
        return errorAttributes;
    }
    ```

    

  - BasicErrorController：

    ![BasicErrorController](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/SpringBoot/06BasicErrorController.png)

    ```java
    @RequestMapping(produces = {"text/html"})//产生html类型的数据；浏览器发送的请求来到这个方法处理
    public ModelAndView errorHtml(HttpServletRequest request, HttpServletResponse response) {
        HttpStatus status = this.getStatus(request);
        Map<String, Object> model = Collections.unmodifiableMap(this.getErrorAttributes(request, this.isIncludeStackTrace(request, MediaType.TEXT_HTML)));
        response.setStatus(status.value());
        
        // 去哪个页面作为错误页面；包含页面地址和页面内容
        ModelAndView modelAndView = this.resolveErrorView(request, response, status, model);
        return modelAndView != null ? modelAndView : new ModelAndView("error", model);
    }
    
    //产生json类型的数据，其他客户端来到这个方法处理
    @RequestMapping
    public ResponseEntity<Map<String, Object>> error(HttpServletRequest request) {
        Map<String, Object> body = this.getErrorAttributes(request, this.isIncludeStackTrace(request, MediaType.ALL));
        HttpStatus status = this.getStatus(request);
        return new ResponseEntity(body, status);
    }
    ```

    处理默认的/error请求

  - ErrorMvcAutoConfiguration中的

    - ErrorPageCustomizer

      ```java
      @Value("${error.path:/error}")
      private String path = "/error";
      ```

      系统出现错误以后来到error请求进行处理；

      相当于以前web.xml注册的错误页面规则。

    - PreserveErrorControllerTargetClassPostProcessor

  - WhitelabelErrorViewConfiguration

    - View
    - BeanNameViewResolver

  - DefaultErrorViewResolverConfiguration

    - DefaultErrorViewResolver

      ```java
      public ModelAndView resolveErrorView(HttpServletRequest request, HttpStatus status, Map<String, Object> model) {
          ModelAndView modelAndView = this.resolve(String.valueOf(status.value()), model);
          if (modelAndView == null && SERIES_VIEWS.containsKey(status.series())) {
              modelAndView = this.resolve((String)SERIES_VIEWS.get(status.series()), model);
          }
      
          return modelAndView;
      }
      
      private ModelAndView resolve(String viewName, Map<String, Object> model) {
          // 默认SpringBoot可以去找到一个页面，error/404
          String errorViewName = "error/" + viewName;
          // 如果模板引擎能起作用，解析这个页面地址，就用模板引擎解析
          TemplateAvailabilityProvider provider = this.templateAvailabilityProviders.getProvider(errorViewName, this.applicationContext);
          // 模板引擎可用的情况下返回到errorViewName指定的视图地址，如果不可用调用下面的resolveResource方法，会在静态资源文件夹下找errorViewName对应的页面 error/404.html
          return provider != null ? new ModelAndView(errorViewName, model) : this.resolveResource(errorViewName, model);
      }
      
      private ModelAndView resolveResource(String viewName, Map<String, Object> model) {
          String[] var3 = this.resourceProperties.getStaticLocations();
          int var4 = var3.length;
      
          for(int var5 = 0; var5 < var4; ++var5) {
              String location = var3[var5];
      
              try {
                  Resource resource = this.applicationContext.getResource(location);
                  resource = resource.createRelative(viewName + ".html");
                  if (resource.exists()) {
                      return new ModelAndView(new DefaultErrorViewResolver.HtmlResourceView(resource), model);
                  }
              } catch (Exception var8) {
              }
          }
      
          return null;
      }
      ```
      
      

#### 执行步骤

  - 一旦系统出现4xx或者5xx之类的错误；ErrorPageCustomizer会生效（定制错误的响应规则）。就会来到`/error`请求；

  - 就会被**BasicErrorController**处理：

    - 响应页面：去哪个页面是由**DefaultErrorViewResolver**来解析得到的。

      ```java
      protected ModelAndView resolveErrorView(HttpServletRequest request, HttpServletResponse response, HttpStatus status, Map<String, Object> model) {
          Iterator var5 = this.errorViewResolvers.iterator();
      
          ModelAndView modelAndView;
          
          // 所有的ErrorViewResolver得到ModelAndView
          do {
              if (!var5.hasNext()) {
                  return null;
              }
      
              ErrorViewResolver resolver = (ErrorViewResolver)var5.next();
              modelAndView = resolver.resolveErrorView(request, status, model);
          } while(modelAndView == null);
      
          return modelAndView;
      }
      ```



#### 如何定制错误响应

  1. 如何定制错误的页面

     - 有模板引擎的情况下：
       - error/状态码[将错误页面命名为 错误状态码.html放在模板引擎文件夹里面的error文件夹下]，发生此状态码的错误就会来到对应的页面；
       - 我们可以使用4xx和5xx作为错误页面的文件名来匹配这种类型的所有错误，精确优先，优先寻找精确的 状态码.html；
       - 页面能获取的信息（**DefaultErrorAttributes** 的 getErrorAttributes方法知悉）：
         1. timestamp：时间戳
         2. status：状态码
         3. error：错误提示
         4. exception：异常对象
         5. message：异常消息
         6. errors：JSR303数据校验的错误都在这里
         7. trace：异常堆栈信息
         8. path：访问路径信息
     - 没有模板引擎（模板引擎找不到这个错误页面），静态资源文件夹下找，只不过没得取出错误信息等。
     - 以上都没有错误页面，就是默认来到SpringBoot默认的错误提示页面；

  2. 如何定制错误的json数据

     - 自定义异常处理&返回定制json数据

       ```JAVA
       @ControllerAdvice
       public class MyExceptionHandler {
       	
           // 1.浏览器和客户端都是返回json，不自适应
           @ResponseBody
           @ExceptionHandler(UserNotExistException.class)
           public Map<String, Object> handleException(Exception e){
               Map<String, Object> map = new HashMap<>();
               map.put("code","user.notexist");
               map.put("message",e.getMessage());
               return map;
           }
       
       }
       ```

       缺点是没有自适应效果，即浏览器返回页面，其他客户端返回json类型，这里是统一返回了json类型的数据。

     - 改进：转发给BaicErrorController的 /error请求来处理，达到自适应的效果

       ```java
       @ControllerAdvice
       public class MyExceptionHandler {
           // 2.转发给BaicErrorController的 /error请求来处理
           @ExceptionHandler(UserNotExistException.class)
           public String handleException(Exception e) {
               Map<String, Object> map = new HashMap<>();
               map.put("code", "user.notexist");
               map.put("message", e.getMessage());
               // 转发到/error
               return "forward:/error";
           }
       }
       ```

       但是还是存在问题，转发到了默认的页面，并没有用到我们的error目录下的页面。原因是因为错误状态码不对，是4xx或5xx，而现在是200状态码，所以需要我们传入自己的状态码。

       ```java
       // 2.转发给BaicErrorController的 /error请求来处理
       @ExceptionHandler(UserNotExistException.class)
       public String handleException(Exception e, HttpServletRequest request) {
           Map<String, Object> map = new HashMap<>();
           // 传入我们自己的错误状态码 4xx 5xx
           request.setAttribute("javax.servlet.error.status_code",500);
           map.put("code", "user.notexist");
           map.put("message", e.getMessage());
           // 转发到/error
           return "forward:/error";
       }
       ```

       **切记错误状态码传入的时候必须要是int类型，传字符串会报错，无法转换为Integer**-->原因如下：

       ```java
       protected HttpStatus getStatus(HttpServletRequest request) {
           Integer statusCode = (Integer)request.getAttribute("javax.servlet.error.status_code");
       ```

       但是还是存在新问题：map传出的数据并没有用上，这时候岂不是回到最原始的状态？

     - 再次改进：需要自适应，又要自定义数据

       出现错误以后，会来到/error请求，会被BasicErrorController处理，这个控制器做了自适应效果，响应出去可以获取的数据是由getErrorAttributes方法（是BasicErrorController的父类AbstractErrorController（ErrorController）规定的方法）得到的，而BasicErrorController的使用前提是，容器中不存在ErrorController这个组件。所以有以下方案：

       1. 完全来编写一个ErrorController的实现类【或者AbstractErrorController的子类】，放在容器中；（**太复杂**）

       2. 页面上能用的数据，或者是json返回能用的数据都是通过errorAttributes.getErrorAttributes得到的；

          容器中DefaultErrorAttributes.getErrorAttributes默认来进行数据处理，也是容器中没有，才用这个默认的DefaultErrorAttributes默认进行数据处理的。那么我们可以**自己写一个组件继承DefaultErrorAttributes然后改写他的getErrorAttributes即可**，如下：

          ```JAVA
          @Component
          public class MyErrorAttributes extends DefaultErrorAttributes {
          
              @Override
              public Map<String, Object> getErrorAttributes(WebRequest webRequest, boolean includeStackTrace) {
                  Map<String, Object> errorAttributes = super.getErrorAttributes(webRequest, includeStackTrace);
                  errorAttributes.put("company","tomxwd");
                  return errorAttributes;
              }
          }
          ```

          这个时候发现，那我怎么在处理异常的时候，也自定义一些数据抛出呢？**这时候我们可以规定一个key作为统一参数，如ext，在处理异常的时候，封装的map，以key为ext的形式setAttribute到request域中去，然后在自定义的MyErrorAttribute中的WebRequest获取出来，再丢到真正抛出去的Map中**，代码如下：

          首先改写ExceptionHandler：

          ```java
          @ExceptionHandler(UserNotExistException.class)
          public String handleException(Exception e, HttpServletRequest request) {
              Map<String, Object> map = new HashMap<>();
              // 传入我们自己的错误状态码 4xx 5xx
              request.setAttribute("javax.servlet.error.status_code",500);
              map.put("code", "user.notexist");
              map.put("message", e.getMessage());
              request.setAttribute("ext",map);
              // 转发到/error
              return "forward:/error";
          }
          ```

          关注`request.setAttribute("ext",map);`这句，然后再到自定义的MyErrorAttributes中进行getAttribute，再塞到响应的map中：

          ```java
          @Component
          public class MyErrorAttributes extends DefaultErrorAttributes {
          
              // 返回值的map就是页面和json能获取的所有字段
              @Override
              public Map<String, Object> getErrorAttributes(WebRequest webRequest, boolean includeStackTrace) {
                  Map<String, Object> errorAttributes = super.getErrorAttributes(webRequest, includeStackTrace);
                  errorAttributes.put("company","tomxwd");
                  Object ext = webRequest.getAttribute("ext",WebRequest.SCOPE_REQUEST);
                  errorAttributes.put("ext",ext);
                  return errorAttributes;
              }
          }
          ```

          这样，在前台页面就可以通过ext获得你拓展的自定义数据，再用ext.key来获取你传递的值，达到目的。

          ![自定义返回json信息](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/SpringBoot/06%E8%87%AA%E5%AE%9A%E4%B9%89%E8%BF%94%E5%9B%9Ejson%E4%BF%A1%E6%81%AF.png)

          可看到company是我们在定义MyErrorAttributes的时候定制的数据，而ext则是各个异常传出来的拓展数据。