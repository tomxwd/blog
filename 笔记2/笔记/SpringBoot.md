# Web开发-错误处理原理&定制错误页面

默认效果：

- 浏览器：返回一个默认的错误页面，显示时间，错误码及类型。![1563322148465](C:\Users\weidou.xie\Desktop\笔记\图\1563322148465.png)

  浏览器发送请求的请求头：

  ![1563326239900](C:\Users\weidou.xie\Desktop\笔记\图\1563326239900.png)

- 如果是其他客户端，通过postman测试，发现默认返回的是json数据。![1563323213638](C:\Users\weidou.xie\Desktop\笔记\图\1563323213638.png)

  其他客户端发送请求的请求头：

  ![1563326381391](C:\Users\weidou.xie\Desktop\笔记\图\1563326381391.png)

原理：

- 点开Maven依赖找到springboot自动配置依赖

  ![1563325144057](C:\Users\weidou.xie\Desktop\笔记\图\1563325144057.png)

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

    ![1563325959651](C:\Users\weidou.xie\Desktop\笔记\图\1563325959651.png)

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

      

- 步骤：

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

      

- 要如何定制错误响应

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

          ![1563334462626](C:\Users\weidou.xie\Desktop\笔记\图\1563334462626.png)

          可看到company是我们在定义MyErrorAttributes的时候定制的数据，而ext则是各个异常传出来的拓展数据。

     

     

     

# 配置嵌入式Servlet容器

SpringBoot默认使用的是嵌入式的Servelt（Tomcat）；

![1563341891221](C:\Users\weidou.xie\Desktop\笔记\图\1563341891221.png)

问题：

1. 如何定制和修改Servlet容器的相关配置；

   - 修改和server有关的配置即可(ServerProperties文件定义)

     ```properties
     server.servlet.context-path=/tomxwd
     server.port=8080
     
     server.tomcat.uri-encoding=UTF-8
     
     # 通用的Servlet容器设置
     server.xxx
     # Tomcat的设置
     server.tomcat.xxx
     ```

   - 编写一个EmbeddedServletContainerCustomizer（**SpringBoot2.x版本为WebServerFactoryCustomizer替代**）：嵌入式的Servlet容器的定制器，来修改Servlet容器的配置。

     ```java
     @Bean
     public WebServerFactoryCustomizer<ConfigurableWebServerFactory> webServerFactoryCustomizer(){
         return new WebServerFactoryCustomizer<ConfigurableWebServerFactory>() {
     
             // 定制嵌入式的Servlet容器相关规则
             @Override
             public void customize(ConfigurableWebServerFactory factory) {
                 factory.setPort(8888);
             }
         };
     }
     ```

     这里是2.x版本的，1.x版本的是注入EmbeddedServletContainerCustomizer，且没有泛型，直接使用container.setPort()即可。

     在1.x中ServerProperties是直接实现了EmbeddedServletContainerCustomizer，所以二者走的底层是一样的。

     **在SpringBoot中会有很多XxxCustomizer帮助我们进行定制配置。**

2. 如何**注册Servlet、Filter、Listene**r三大组件，以往Web项目中会有/webapp/WEB-INF/web.xml提供注册，现在用嵌入式的Servlet如何进行注册。

   由于SpringBoot默认是以jar包的方式启动嵌入式的Servlet容器来启动SpringBoot的Web应用，没有web.xml文件。所以注册三大组件用以下的方式。

   - ServletRegistrationBean

     编写Servlet

     ```java
     public class MyServlet extends HttpServlet {
     
         // 处理get请求
         @Override
         protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
             doPost(req,resp);
         }
     
         // 处理post请求
         @Override
         protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
             resp.getWriter().write("Hello MyServlet");
         }
     }
     ```

     注册到Spring容器中：

     ```JAVA
     // 注册三大组件
     // 1. Servlet
     @Bean
     public ServletRegistrationBean<MyServlet> myServlet(){
         ServletRegistrationBean<MyServlet> servletServletRegistrationBean = new ServletRegistrationBean<>(new MyServlet(),"/myServlet");
         servletServletRegistrationBean.setLoadOnStartup(1);
         return servletServletRegistrationBean;
     }
     ```

     最后通过访问/myServlet可以看到页面输出“Hello MyServlet”证明已成功注册。

   - FilterRegistrationBean

     编写Filter：

     ```java
     public class MyFilter implements Filter {
     
         @Override
         public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
             System.out.println("MyFilter process...");
             filterChain.doFilter(servletRequest,servletResponse);
         }
     
         @Override
         public void init(FilterConfig filterConfig) throws ServletException {
             System.out.println("MyFilter初始化");
         }
     
         @Override
         public void destroy() {
             System.out.println("MyFilter销毁");
         }
     }
     ```

     注册到Spring容器中：

     ```java
     // 2. Filter
     @Bean
     public FilterRegistrationBean<MyFilter> filterFilterRegistrationBean(){
         FilterRegistrationBean<MyFilter> registrationBean = new FilterRegistrationBean<>(new MyFilter());
         registrationBean.setUrlPatterns(Arrays.asList("/hello","/myServlet"));
         return registrationBean;
     }
     ```

     在访问`/hello，/myServlet`的时候回进行过滤操作，输出`MyFilter process...`。

   - ServletListenerRegistrationBean

     编写Listener：

     ```java
     public class MyListener implements ServletContextListener {
         @Override
         public void contextInitialized(ServletContextEvent sce) {
             System.out.println("contextInitialized...web应用启动");
         }
     
         @Override
         public void contextDestroyed(ServletContextEvent sce) {
             System.out.println("contextDestroyed...当前web应用销毁");
         }
     }
     ```

     注册到Spring容器中：

     ```java
     // 3. Listener
     @Bean
     public ServletListenerRegistrationBean<MyListener> servletListenerRegistrationBean(){
         ServletListenerRegistrationBean<MyListener> listenerRegistrationBean = new ServletListenerRegistrationBean<>();
         listenerRegistrationBean.setListener(new MyListener());
         return listenerRegistrationBean;
     }
     ```

     观察控制台，在项目启动的时候，会输出`contextInitialized...web应用启动`；正常退出项目（Exit）的时候，会输出`contextDestroyed...当前web应用销毁`；

   最好的例子是SpringBoot在帮我们自动配置SpringMVC的时候，自动注册了SpringMVC的前端控制器（DispatcherServlet）；

   ```java
   @Bean(name = {"dispatcherServletRegistration"})
   @ConditionalOnBean(
       value = {DispatcherServlet.class},
       name = {"dispatcherServlet"}
   )
   public DispatcherServletRegistrationBean dispatcherServletRegistration(DispatcherServlet dispatcherServlet) {
       // 默认拦截为/，包括了静态资源，但是不拦截jps请求； /*会拦截jsp
       // 可以通过修改spring.mvc.servlet.path=来修改默认拦截的路径，SpringBoot1.x版本为server.servletPath来修改前端控制器默认拦截的请求路径
       DispatcherServletRegistrationBean registration = new DispatcherServletRegistrationBean(dispatcherServlet, this.webMvcProperties.getServlet().getPath());
       registration.setName("dispatcherServlet");
       registration.setLoadOnStartup(this.webMvcProperties.getServlet().getLoadOnStartup());
       if (this.multipartConfig != null) {
           registration.setMultipartConfig(this.multipartConfig);
       }
   
       return registration;
   }
   ```

   

3. SpringBoot能不能支持其他的Servlet容器；

   - Jetty（长连接）
   - Undertow（不支持JSP）

   ![1563346802686](C:\Users\weidou.xie\Desktop\笔记\图\1563346802686.png)

   SpringBoot默认支持：

   1. Tomcat（默认使用）

      ```xml
      <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-web</artifactId>
      </dependency>
      ```

      **引入Web模块的时候默认就是使用嵌入式Tomcat作为Servlet容器的。

   2. Jetty

   3. UnderTow

   想要切换就两个步骤：

   1. 排除tomcat的start依赖

      ```xml
      <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-web</artifactId>
          <exclusions>
              <exclusion>
                  <artifactId>spring-boot-starter-tomcat</artifactId>
                  <groupId>org.springframework.boot</groupId>
              </exclusion>
          </exclusions>
      </dependency>
      ```

   2. 引入其他容器的starter

      - 切换为Jetty：

        ```xml
        <!-- 引入其他的Servlet容器 -->
        <!-- Jetty -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jetty</artifactId>
        </dependency>
        ```

      - 切换为UnderTow

        ```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-undertow</artifactId>
        </dependency>
        ```

4. 嵌入式Servlet容器自动配置原理

   SpringBoot1.x看`EmbeddedServletContainerAutoConfiguration`类；

   EmbeddedServletContainerAutoConfiguration：

   ```java
   @AutoConfigureOrder(-2147483648)
   @Configuration
   @ConditionalOnWebApplication
   @Import({EmbeddedServletContainerAutoConfiguration.BeanPostProcessorsRegistrar.class})
   // 导入BeanPostProcessorsRegistrar，给容器中导入一些组件
   // 导入了EmbeddedServletContainerCustomizerBeanPostProcessor：后置处理器：bean初始化前后（创建完对象，还没属性赋值）执行初始化工作
   public class EmbeddedServletContainerAutoConfiguration {
       
       
       @Configuration
       @ConditionalOnClass({Servlet.class, Tomcat.class})//判断当前是否引入了tomcat依赖
       @ConditionalOnMissingBean(
           value = {EmbeddedServletContainerFactory.class},
           search = SearchStrategy.CURRENT
       )//这个类会先判断是否有用户自定义的EmbeddedServletContainerFactory:嵌入式的Servlet容器工厂：作用是创建嵌入式的Servlet容器。
       public static class EmbeddedTomcat {
           public EmbeddedTomcat() {
           }
   
           @Bean
           public TomcatEmbeddedServletContainerFactory tomcatEmbeddedServletContainerFactory() {
               return new TomcatEmbeddedServletContainerFactory();
           }
       }
   ```

   - EmbeddedServletContainerFactory（嵌入式Servlet容器工厂）:

     ```java
     public interface EmbeddedServletContainerFactory{
         // 获取嵌入式的Servlet容器 
         EmbeddedServletContainer getEmbeddedServletContainer(ServletContextInitalizer...initalizers)
     }
     ```

     ![1563349058727](C:\Users\weidou.xie\Desktop\笔记\图\1563349058727.png)

   - EmbeddedServletContainer（嵌入式的Servlet容器）

     ![1563349164280](C:\Users\weidou.xie\Desktop\笔记\图\1563349164280.png)

   - 以嵌入式的Tomcat容器工厂（**TomcatEmbeddedServletContainerFactory**）为例

     ```java
     @Override
     public EmbeddedServletContainer getEmbeddedServletContainer(
           ServletContextInitializer... initializers) {
         // 创建一个Tomcat
        Tomcat tomcat = new Tomcat();
         
         // 配置Tomcat的基本环境
        File baseDir = (this.baseDirectory != null) ? this.baseDirectory
              : createTempDir("tomcat");
        tomcat.setBaseDir(baseDir.getAbsolutePath());
         // 连接器
        Connector connector = new Connector(this.protocol);
        tomcat.getService().addConnector(connector);
        customizeConnector(connector);
        tomcat.setConnector(connector);
        tomcat.getHost().setAutoDeploy(false);
         // 引擎
        configureEngine(tomcat.getEngine());
        for (Connector additionalConnector : this.additionalTomcatConnectors) {
           tomcat.getService().addConnector(additionalConnector);
        }
        prepareContext(tomcat.getHost(), initializers);
         
         // 将配置好的Tomcat传入进去，返回一个EmbeddedServletContainer嵌入式的Servlet容器；并且启动Tomcat容器
        return getTomcatEmbeddedServletContainer(tomcat);
     }
     ```

   - 我们对嵌入式容器的配置修改是怎么生效的？

     1. 修改ServerProperties里的东西
     2. EmbeddedServletContainerCustomizer嵌入式Servlet容器的定制器

     **EmbeddedServletContainerCustomizer**：定制器帮我们修改了Servlet容器的配置，原理如下：

     容器中也导入了**EmbeddedServletContainerCustomizerBeanPostProcessor**：

     ```java
     // 初始化之前
     @Override
     public Object postProcessBeforeInitialization(Object bean, String beanName)
           throws BeansException {
         // 如果当前初始化的是ConfigurableEmbeddedServletContainer这个类型的组件，就调用初始化方法
        if (bean instanceof ConfigurableEmbeddedServletContainer) {
           postProcessBeforeInitialization((ConfigurableEmbeddedServletContainer) bean);
        }
        return bean;
     }
     
     private void postProcessBeforeInitialization(
         ConfigurableEmbeddedServletContainer bean) {
         // 获取所有的定制器，调用每一个定制器的customize方法来给Servlet容器进行属性赋值
         for (EmbeddedServletContainerCustomizer customizer : getCustomizers()) {
             customizer.customize(bean);
         }
     }
     
     private Collection<EmbeddedServletContainerCustomizer> getCustomizers() {
         if (this.customizers == null) {
             // Look up does not include the parent context
             // 只有一个核心：从容器中获取所有这个类型的组件：EmbeddedServletContainerCustomizer
             // 得到结论：想要定制Servlet容器，给容器中添加一个EmbeddedServletContainerCustomizer类型的组件，而ServerProperties也是这个组件类型。 
             this.customizers = new ArrayList<EmbeddedServletContainerCustomizer>(
                 this.beanFactory
                 .getBeansOfType(EmbeddedServletContainerCustomizer.class,
                                 false, false)
                 .values());
             Collections.sort(this.customizers, AnnotationAwareOrderComparator.INSTANCE);
             this.customizers = Collections.unmodifiableList(this.customizers);
         }
         return this.customizers;
     }
     ```

   - 步骤

     1. SpringBoot根据导入的依赖情况，给容器中添加相应的`EmbeddedServletContainerFactory`如【`TomcatEmbeddedServletContainerFactory`】

     2. 容器中某个组件要创建对象就会惊动后置处理器（`EmbeddedServletContainerCustomizerBeanPostProcessor`）进行工作；

        只要是嵌入式的Servlet容器工厂，后置处理器就工作。

     3. 后置处理器就会从容器中获取所有的`EmbeddedServletContainerCustomizer`调用定制器的定制方法

   

   

   

   ---

   SpringBoot2.x看`EmbeddedWebServerFactoryCustomizerAutoConfiguration`类；

   `EmbeddedWebServerFactoryCustomizerAutoConfiguration`：嵌入式的Servlet容器自动配置：

   ```java
   @Configuration
   @ConditionalOnWebApplication
   @EnableConfigurationProperties({ServerProperties.class})
   public class EmbeddedWebServerFactoryCustomizerAutoConfiguration {
       public EmbeddedWebServerFactoryCustomizerAutoConfiguration() {
       }
   
       @Configuration
       @ConditionalOnClass({HttpServer.class})
       public static class NettyWebServerFactoryCustomizerConfiguration {
           public NettyWebServerFactoryCustomizerConfiguration() {
           }
   
           @Bean
           public NettyWebServerFactoryCustomizer nettyWebServerFactoryCustomizer(Environment environment, ServerProperties serverProperties) {
               return new NettyWebServerFactoryCustomizer(environment, serverProperties);
           }
       }
   
       @Configuration
       @ConditionalOnClass({Undertow.class, SslClientAuthMode.class})// 判断当前是否引入了Undertow依赖
       public static class UndertowWebServerFactoryCustomizerConfiguration {
           public UndertowWebServerFactoryCustomizerConfiguration() {
           }
   
           @Bean
           public UndertowWebServerFactoryCustomizer undertowWebServerFactoryCustomizer(Environment environment, ServerProperties serverProperties) {
               return new UndertowWebServerFactoryCustomizer(environment, serverProperties);
           }
       }
   
       @Configuration
       @ConditionalOnClass({Server.class, Loader.class, WebAppContext.class})// 判断当前是否引入了jetty依赖
       public static class JettyWebServerFactoryCustomizerConfiguration {
           public JettyWebServerFactoryCustomizerConfiguration() {
           }
   
           @Bean
           public JettyWebServerFactoryCustomizer jettyWebServerFactoryCustomizer(Environment environment, ServerProperties serverProperties) {
               return new JettyWebServerFactoryCustomizer(environment, serverProperties);
           }
       }
   
       @Configuration
       @ConditionalOnClass({Tomcat.class, UpgradeProtocol.class})//判断当前是否引入了tomcat依赖
       public static class TomcatWebServerFactoryCustomizerConfiguration {
           public TomcatWebServerFactoryCustomizerConfiguration() {
           }
   
           @Bean
           public TomcatWebServerFactoryCustomizer tomcatWebServerFactoryCustomizer(Environment environment, ServerProperties serverProperties) {
               return new TomcatWebServerFactoryCustomizer(environment, serverProperties);
           }
       }
   }
   ```

   ***未进行探究（待续！）***

   ---





4. 嵌入式Servlet容器启动原理

   什么时候创建嵌入式的Servlet容器工厂，什么时候获取嵌入式Servlet容器并启动Tomcat；

   获取嵌入式的Servlet容器工厂：

   1. SpringBoot应用启动运行run方法

   2. `refreshContext(context)`：SpringBoot刷新ioc容器【创建ioc容器对象，并初始化容器，创建容器中的每一个组件】；如果是Web应用创建`AnnotationConfigEmbeddedWebApplicationContext`否则创建`AnnotationConfigApplicationContext`;

   3. `refresh(context)`:刷新刚才创建好的ioc容器；

      ```java
      @Override
      public void refresh() throws BeansException, IllegalStateException {
         synchronized (this.startupShutdownMonitor) {
            // Prepare this context for refreshing.
            prepareRefresh();
      
            // Tell the subclass to refresh the internal bean factory.
            ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
      
            // Prepare the bean factory for use in this context.
            prepareBeanFactory(beanFactory);
      
            try {
               // Allows post-processing of the bean factory in context subclasses.
               postProcessBeanFactory(beanFactory);
      
               // Invoke factory processors registered as beans in the context.
               invokeBeanFactoryPostProcessors(beanFactory);
      
               // Register bean processors that intercept bean creation.
               registerBeanPostProcessors(beanFactory);
      
               // Initialize message source for this context.
               initMessageSource();
      
               // Initialize event multicaster for this context.
               initApplicationEventMulticaster();
      
               // Initialize other special beans in specific context subclasses.
               onRefresh();
      
               // Check for listener beans and register them.
               registerListeners();
      
               // Instantiate all remaining (non-lazy-init) singletons.
               finishBeanFactoryInitialization(beanFactory);
      
               // Last step: publish corresponding event.
               finishRefresh();
            }
      
            catch (BeansException ex) {
               if (logger.isWarnEnabled()) {
                  logger.warn("Exception encountered during context initialization - " +
                        "cancelling refresh attempt: " + ex);
               }
      
               // Destroy already created singletons to avoid dangling resources.
               destroyBeans();
      
               // Reset 'active' flag.
               cancelRefresh(ex);
      
               // Propagate exception to caller.
               throw ex;
            }
      
            finally {
               // Reset common introspection caches in Spring's core, since we
               // might not ever need metadata for singleton beans anymore...
               resetCommonCaches();
            }
         }
      }
      ```

   4. 在`onRefresh();`的时候，Web的ioc容器（EmbeddedWebApplicationContext）重写了onRefresh方法

      ```java
      @Override
      protected void onRefresh() {
         super.onRefresh();
         try {
            createEmbeddedServletContainer();
         }
         catch (Throwable ex) {
            throw new ApplicationContextException("Unable to start embedded container",
                  ex);
         }
      }
      ```

   5. Web的ioc容器会创建嵌入式的Servlet容器：**createEmbeddedServletContainer();**方法；

      ```java
      private void createEmbeddedServletContainer() {
         EmbeddedServletContainer localContainer = this.embeddedServletContainer;
         ServletContext localServletContext = getServletContext();
         if (localContainer == null && localServletContext == null) {
            EmbeddedServletContainerFactory containerFactory = getEmbeddedServletContainerFactory();
            this.embeddedServletContainer = containerFactory
                  .getEmbeddedServletContainer(getSelfInitializer());
         }
         else if (localServletContext != null) {
            try {
               getSelfInitializer().onStartup(localServletContext);
            }
            catch (ServletException ex) {
               throw new ApplicationContextException("Cannot initialize servlet context",
                     ex);
            }
         }
         initPropertySources();
      }
      ```

   6. 获取嵌入式的Servlet容器工厂：

      ```java
      EmbeddedServletContainerFactory containerFactory = getEmbeddedServletContainerFactory();
      ```

      在ioc中获取：

      ```java
      String[] beanNames = getBeanFactory()
            .getBeanNamesForType(EmbeddedServletContainerFactory.class);
      ```

      从ioc容器中获取EmbeddedServletContainerFactory【嵌入式Servlet容器工厂】组件，如【TomcatEmbeddedServletContainerFactory】创建对象，后置处理器一看是这个对象，就获取所有的定制器来先定制Servlet容器的相关配置；

   7. **使用容器工厂获取嵌入式的Servlet容器：**

      ```java
      this.embeddedServletContainer = containerFactory
            .getEmbeddedServletContainer(getSelfInitializer());
      ```

   8. 嵌入式的Servlet容器创建对象并启动Servlet容器；【tomcat启动（this.tomcat.start()）】

      **先启动嵌入式的Servlet容器，再将ioc容器中剩下没有创建出来的对象获取出来；**

   9. ***总结：*****ioc容器启动创建嵌入式的Servlet容器**



5. 使用外置的Servlet容器

   - 嵌入式Servlet容器：

     应用打成可执行的jar包；

     ​	**优点：**简单、便捷

     ​	**缺点：**默认不支持JSP、优化定制比较复杂（使用定制器【ServerProperties】、【自定义EmbeddedServletContainerCustomizer】、【自己编写嵌入式Servlet容器的创建工厂EmbeddedServletContainerFactory】）；

   - 外置的Servlet容器：

     外面安装Tomcat----应用war包的方式打包；

     建项目的时候选择war包形式，然后要用tomcat来启动，注意版本搭配。

     步骤：

     1. 创建一个war项目（利用idea创建好目录结构）

     2. 将嵌入式的tomcat指定为provided

        ```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
            <scope>provided</scope>
        </dependency>
        ```

     3. 必须编写一个**SpringBootServletInitializer**的子类，并调用configure方法

        ```java
        public class ServletInitializer extends SpringBootServletInitializer {
        
            @Override
            protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
                // 传入SpringBoot应用的主程序
                return application.sources(SpringBootMvcWarApplication.class);
            }
        
        }
        ```

     4. 启动服务器即可。

   - **原理：**

     jar包：执行SpringBoot主类的main方法，启动ioc容器，创建嵌入式的Servlet容器；

     war包：启动服务器，然后**服务器启动SpringBoot应用**【SpringBootServletInitializer】，启动ioc容器；

     

     servlet3.0规范：

     **规则：**

     1. 服务器启动（web应用启动）会创建当前web应用里面的每一个jar包里面的ServletContainerInitalizer的实例
     2. ServletContainerInitalizer的实现放在jar包的META-INF/services文件夹下，有一个名为javax.servlet.ServletContainerInitializer的文件，内容就是ServletContainerInitalizer的实现类的全类名。
     3. 还可以使用**@HandlesTypes**注解，在应用启动的时候加载我们感兴趣的类。

     

     **流程：**

     1. 启动Tomcat

     2. org\springframework\spring-web\5.1.8.RELEASE\spring-web-5.1.8.RELEASE.jar!\META-INF\services\javax.servlet.ServletContainerInitializer：

        Spring的web模块里面有这个文件，内容就是`org.springframework.web.SpringServletContainerInitializer`说明在应用启动的时候要启动他

     3. SpringServletContainerInitializer将@HandlesTypes(WebApplicationInitializer.class)标注的所有这个类型的类都传入到onStartup方法的Set<Class<?>>;为这些（WebApplicationInitializer）类型的类创建实例；

     4. 每一个WebApplicationInitializer都调用自己的onStartup方法。

        ![1563355527240](C:\Users\weidou.xie\Desktop\笔记\图\1563355527240.png)

     5. 相当于我们的SpringBootServletInitalizer的类会被创建对象，并执行onStartup方法

     6. SpringBootServletInitalizer实例执行onStartup的时候会createRootApplicationContext；创建容器

        ```java
        protected WebApplicationContext createRootApplicationContext(ServletContext servletContext) {
            // 1、 创建SpringApplicationBuilder（spring应用的构建器）
            SpringApplicationBuilder builder = this.createSpringApplicationBuilder();
            builder.main(this.getClass());
            ApplicationContext parent = this.getExistingRootWebApplicationContext(servletContext);
            if (parent != null) {
                this.logger.info("Root context already created (using as parent).");
                servletContext.setAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE, (Object)null);
                builder.initializers(new ApplicationContextInitializer[]{new ParentContextApplicationContextInitializer(parent)});
            }
        
            builder.initializers(new ApplicationContextInitializer[]{new ServletContextApplicationContextInitializer(servletContext)});
            builder.contextClass(AnnotationConfigServletWebServerApplicationContext.class);
            // 2、 调用configure方法，子类重写了这个方法，将SpringBoot的主程序类传进来。
            builder = this.configure(builder);
            builder.listeners(new ApplicationListener[]{new SpringBootServletInitializer.WebEnvironmentPropertySourceInitializer(servletContext)});
            // 3、 使用builder创建一个Spring应用
            SpringApplication application = builder.build();
            if (application.getAllSources().isEmpty() && AnnotationUtils.findAnnotation(this.getClass(), Configuration.class) != null) {
                application.addPrimarySources(Collections.singleton(this.getClass()));
            }
        
            Assert.state(!application.getAllSources().isEmpty(), "No SpringApplication sources have been defined. Either override the configure method or add an @Configuration annotation");
            if (this.registerErrorPageFilter) {
                application.addPrimarySources(Collections.singleton(ErrorPageFilterConfiguration.class));
            }
        	// 4、 启动Spring
            return this.run(application);
        }
        ```

     7. Spring的应用就启动并创建IOC容器。

        ```java
        public ConfigurableApplicationContext run(String... args) {
            StopWatch stopWatch = new StopWatch();
            stopWatch.start();
            ConfigurableApplicationContext context = null;
            Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList();
            this.configureHeadlessProperty();
            SpringApplicationRunListeners listeners = this.getRunListeners(args);
            listeners.starting();
        
            Collection exceptionReporters;
            try {
                ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
                ConfigurableEnvironment environment = this.prepareEnvironment(listeners, applicationArguments);
                this.configureIgnoreBeanInfo(environment);
                Banner printedBanner = this.printBanner(environment);
                context = this.createApplicationContext();
                exceptionReporters = this.getSpringFactoriesInstances(SpringBootExceptionReporter.class, new Class[]{ConfigurableApplicationContext.class}, context);
                this.prepareContext(context, environment, listeners, applicationArguments, printedBanner);
                // 刷新IOC容器
                this.refreshContext(context);
                this.afterRefresh(context, applicationArguments);
                stopWatch.stop();
                if (this.logStartupInfo) {
                    (new StartupInfoLogger(this.mainApplicationClass)).logStarted(this.getApplicationLog(), stopWatch);
                }
        
                listeners.started(context);
                this.callRunners(context, applicationArguments);
            } catch (Throwable var10) {
                this.handleRunFailure(context, var10, exceptionReporters, listeners);
                throw new IllegalStateException(var10);
            }
        
            try {
                listeners.running(context);
                return context;
            } catch (Throwable var9) {
                this.handleRunFailure(context, var9, exceptionReporters, (SpringApplicationRunListeners)null);
                throw new IllegalStateException(var9);
            }
        }
        ```

     **总结启动原理：**

     - Servlet3.0标准ServletContainerInitializer扫描的所有jar包中的META-INF/services/javax.servlet.ServletContainerInitializer文件指定的类并加载这个类
     - 加载spring web包下的SpringServletContainerInitializer
     - 扫描@HandleType(WebApplicationInitializer)
     - 加载SpringBootServletInitializer并运行onStartup方法
     - 加载@SpringBootApplication主类，启动容器等。

     总的来说就是先启动Servlet容器，再启动SpringBoot应用。

     

     

     

# SpringBoot与Docker

![1563356733916](C:\Users\weidou.xie\Desktop\笔记\图\1563356733916.png)

1. 简介：

   Docker是一个开源的应用容器引擎；

   Docker支持将软件编译成一个镜像，然后在镜像中各种软件做好配置，将镜像发布出去，其他使用者可以直接使用这个镜像；

   运行中的这个镜像成为容器，容器的启动是非常快速的。



## Docker核心概念

![1563356808404](C:\Users\weidou.xie\Desktop\笔记\图\1563356808404.png)

docker主机（Host）：安装了Docker程序的机器（Docker直接安装在操作系统之上）

docker客户端（Client）：连接docker主机进行操作；

docker仓库（Registry）：用来保存各种打包好的软件镜像；

docker镜像（Images）：软件打包好的镜像，放在docker仓库中；

docker容器（Container）：镜像启动后的实例成为一个容器；容器是独立运行的一个或一组应用；

使用Docker的步骤：

1. 安装Docker
2. 去Docker的仓库找到这个软件对应的镜像
3. 使用Docker运行这个镜像，这个镜像就会生成一个Docker容器
4. 对容器的启动停止就是对软件的启动停止



## 安装Docker

#### 安装Linux虚拟机

1. VMware（太重量级了）
2. **VirtualBox**（Oracle免费的虚拟机软件，小巧）

这里用VirtualBox，点开安装完，直接导入虚拟机文件（ova格式），点上重置网卡选项。

然后安装SmarTTY客户端软件。

此时还没能连上虚拟机，需要**设置虚拟机的网络**

1. 桥接网络
2. 选好网卡
3. 点上接入网线
4. service network restart（centos 7）或重启虚拟机
5. 查看虚拟机IP地址（`ip addr`）

使用SmarTTY连接即可使用。

#### 安装Docker

![1563408764502](C:\Users\weidou.xie\Desktop\笔记\图\1563408764502.png)

//TODO













![1563409124079](C:\Users\weidou.xie\Desktop\笔记\图\1563409124079.png)



#### 容器操作

软件镜像（类似QQ安装程序）----运行镜像----产生一个容器（正在运行的软件，类比启动QQ.exe）

![1563410783077](C:\Users\weidou.xie\Desktop\笔记\图\1563410783077.png)

步骤：

1. 搜索镜像

   ```shell
   [root@izwz92w1juq9po3gy6wrk7z ~]# docker search tomcat
   ```

2. 拉取镜像

   ```shell
   [root@izwz92w1juq9po3gy6wrk7z ~]# docker pull tomcat
   ```

3. 根据镜像启动容器

   ```shell
   [root@izwz92w1juq9po3gy6wrk7z ~]# docker run --name mytomcat -d tomcat:latest
   ```

4. 查看运行中的容器

   ```shell
   [root@izwz92w1juq9po3gy6wrk7z ~]# docker ps
   ```

   由于没有映射端口，所以访问不到tomcat。

5. 停止运行中的容器

   ```shell
   [root@izwz92w1juq9po3gy6wrk7z ~]# docker stop mytomcat
   ```

   可以用容器id来停止。

6. 查看所有的容器

   ```shell
   [root@izwz92w1juq9po3gy6wrk7z ~]# docker ps -a
   ```

7. 启动容器

   ```shell
   [root@izwz92w1juq9po3gy6wrk7z ~]# docker start tomcat
   ```

8. 删除容器

   ```shell
   [root@izwz92w1juq9po3gy6wrk7z ~]# docker rm mytomcat
   ```

9. 映射端口并运行tomcat容器

   ```shell
   [root@izwz92w1juq9po3gy6wrk7z ~]# docker run --name mytomcat -d -p 8888:8080 tomcat
   ```

   -d：后台运行

   -p：将主机的端口映射到容器的一个端口 				主机端口:容器内部端口

10. 注意：要访问到这个Tomcat，要么就开放指定端口，要么就关了防火墙，否则外部无法访问。

    查看防火墙的状态：

    ```shell
    [root@izwz92w1juq9po3gy6wrk7z ~]# service firewalld status
    ```

    关闭防火墙：

    ```shell
    [root@izwz92w1juq9po3gy6wrk7z ~]# service firewalld stop
    ```

11. 查看docker的日志

    ```shell
    [root@izwz92w1juq9po3gy6wrk7z ~]# docker logs mytomcat
    ```

直接看官方说明文档会有详细的配置说明。



## 环境搭建

![1563411555138](C:\Users\weidou.xie\Desktop\笔记\图\1563411555138.png)



#### 安装mysql示例

```shell
docker pull mysql
```

安装最新的mysql。



这是错误的启动：

```shell
[root@izwz92w1juq9po3gy6wrk7z ~]# docker run --name springboot-mysql -d mysql
47b024bc5914fd5f8b4528d2c921a66289b0952d3aa82e7346e6fc7d4df33a69
```

`docker ps`查看运行中的容器，并没有springboot-mysql

而`docker ps -a`查看所有容器，发现springboot-mysql是exit状态。mysql退出了。



查看docker的日志，用`docker logs springboot-mysql`:

```shell
[root@izwz92w1juq9po3gy6wrk7z ~]# docker logs springboot-mysql
error: database is uninitialized and password option is not specified 
  You need to specify one of MYSQL_ROOT_PASSWORD, MYSQL_ALLOW_EMPTY_PASSWORD and MYSQL_RANDOM_ROOT_PASSWORD
```

错误日志：数据库在初始化的时候密码项没有指定，我们必须指定这里面其中一个参数（`MYSQL_ROOT_PASSWORD  ,  MYSQL_ALLOW_EMPTY_PASSWORD  ,  MYSQL_RANDOM_ROOT_PASSWORD`）。



正确的启动（应该到docker-hub查看官方的文档进行启动）：

```shell
[root@izwz92w1juq9po3gy6wrk7z ~]# docker run --name springboot-mysql -e MYSQL_ROOT_PASSWORD=root -d mysql;
```

官网上看到的比较正确的启动命令。但是发现还差个端口映射没配。

重新整一份：

```shell
[root@izwz92w1juq9po3gy6wrk7z ~]# docker run --name springboot-mysql -p 12389:3306 -e MYSQL_ROOT_PASSWORD=root -d mysql;
```

即可尝试连接：

报错：

![1563413929239](C:\Users\weidou.xie\Desktop\笔记\图\1563413929239.png)

mysql8.0默认使用caching_sha2_password身份验证机制；客户端并不支持新的加密方式

解决方案：

修改用户（root）的加密方式：

1. 进入mysql

   ```shell
   [root@izwz92w1juq9po3gy6wrk7z ~]# docker exec -it springboot-mysql bash
   ```

2. 登录mysql

   ```shell
   root@fe1861f43b33:/# mysql -u root -p     
   Enter password: 
   Welcome to the MySQL monitor.  Commands end with ; or \g.
   Your MySQL connection id is 12
   Server version: 8.0.16 MySQL Community Server - GPL
   
   Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.
   
   Oracle is a registered trademark of Oracle Corporation and/or its
   affiliates. Other names may be trademarks of their respective
   owners.
   
   Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
   
   mysql> 
   ```

3. 设置用户配置项

   1. 查看用户信息

      ```shell
      mysql> select host,user,plugin,authentication_string from mysql.user;
      +-----------+------------------+-----------------------+------------------------------------------------------------------------+
      | host      | user             | plugin                | authentication_string                                                  |
      +-----------+------------------+-----------------------+------------------------------------------------------------------------+
      | %         | root             | caching_sha2_password | $A$005$L-KWV|s}U=<r{rHpTU/NoyY7v1/4n425RQHZqkcYIcqJujE4Z7h4kIfuoA |
      | localhost | mysql.infoschema | caching_sha2_password | $A$005$THISISACOMBINATIONOFINVALIDSALTANDPASSWORDTHATMUSTNEVERBRBEUSED |
      | localhost | mysql.session    | caching_sha2_password | $A$005$THISISACOMBINATIONOFINVALIDSALTANDPASSWORDTHATMUSTNEVERBRBEUSED |
      | localhost | mysql.sys        | caching_sha2_password | $A$005$THISISACOMBINATIONOFINVALIDSALTANDPASSWORDTHATMUSTNEVERBRBEUSED |
      | localhost | root             | caching_sha2_password | $A$005$>gKE+&?.jtS_WFv
                                                                                     oo77e2AccPJtYR3rHsSutMTVg.I4YetUG7w2tWhQmk2 |
      +-----------+------------------+-----------------------+------------------------------------------------------------------------+
      5 rows in set (0.00 sec)
      ```

      这里可以看到：host是%表示不限制ip，localhost表示本机，那么%使用的plugin不是mysql_native_password，则需要改成mysql_native_password。

   2. 修改加密方式并刷新权限

      ```shell
      mysql> ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY 'root';
      Query OK, 0 rows affected (0.01 sec)
      
      mysql> flush privileges;
      Query OK, 0 rows affected (0.00 sec)
      ```

   3. 再次查看用户信息

      ```shell
      mysql> select host,user,plugin,authentication_string from mysql.user;
      +-----------+------------------+-----------------------+------------------------------------------------------------------------+
      | host      | user             | plugin                | authentication_string                                                  |
      +-----------+------------------+-----------------------+------------------------------------------------------------------------+
      | %         | root             | mysql_native_password | *81F5E21E35407D884A6CD4A731AEBFB6AF209E1B                              |
      | localhost | mysql.infoschema | caching_sha2_password | $A$005$THISISACOMBINATIONOFINVALIDSALTANDPASSWORDTHATMUSTNEVERBRBEUSED |
      | localhost | mysql.session    | caching_sha2_password | $A$005$THISISACOMBINATIONOFINVALIDSALTANDPASSWORDTHATMUSTNEVERBRBEUSED |
      | localhost | mysql.sys        | caching_sha2_password | $A$005$THISISACOMBINATIONOFINVALIDSALTANDPASSWORDTHATMUSTNEVERBRBEUSED |
      | localhost | root             | caching_sha2_password | $A$005$>gKE+&?.jtS_WFv
                                                                                     oo77e2AccPJtYR3rHsSutMTVg.I4YetUG7w2tWhQmk2 |
      +-----------+------------------+-----------------------+------------------------------------------------------------------------+
      5 rows in set (0.00 sec)
      ```

      此时，host为%的用户root，plugin为mysql_native_password，再次连接试试。

4. 再次连接

   ![1563414961007](C:\Users\weidou.xie\Desktop\笔记\图\1563414961007.png)

   可行。

   

几个其他高级操作：

指定配置文件：

```shell
$ docker run --name some-mysql -v /my/custom:/etc/mysql/conf.d -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag
```

-v：把主机的/my/custom文件夹挂载到mysql容器的/etc/mysql/conf.d文件夹里面，改mysql的配置文件，就只需要把mysql配置文件放在/my/custom，官方文档说，会将合并里面的文件。



不使用配置文件：

```shell
$ docker run --name some-mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
```

--Xxx 指定配置mysql的一些参数，这里改变字符集配置为utf-8；



我们重新生成一个容器，加上`--character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci`，改变其字符集，方便以后的操作，以免中文乱码：

```shell
[root@izwz92w1juq9po3gy6wrk7z ~]# docker run --name springboot-mysql -d -p 12389:3306 -e MYSQL_ROOT_PASSWORD=root mysql --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
```

此时还是要去改用户root的加密方式，过程如上。



其他的容器安装自己去看docker-hub即可，可以尝试安装。







---

# SpringBoot与数据访问

> 使用1.x版本的SpringBoot

JDBC、Mybatis、Spring Data JPA

![1563415847166](C:\Users\weidou.xie\Desktop\笔记\图\1563415847166.png)



JDBC

![1563415907584](C:\Users\weidou.xie\Desktop\笔记\图\1563415907584.png)

pom.xml

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-jdbc</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <scope>runtime</scope>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

主要是引入了jdbc-starter和mysql驱动。

这次试用yml配置文件：

```yaml
spring:
  datasource:
    username: root
    password: root
    url: jdbc:mysql://ip:12389/jdbc?characterEncoding=utf8&useSSL=false
    driver-class-name: com.mysql.jdbc.Driver
```

测试是否成功：

```java
@Autowired
DataSource dataSource;

@Test
public void contextLoads() throws SQLException {
    System.out.println(dataSource);
    System.out.println(dataSource.getClass());
    Connection conn = dataSource.getConnection();
    System.out.println(conn);
}
```

效果：

​	默认使用org.apache.tomcat.jdbc.pool.DataSource作为数据源；

​	数据源的配置都在DataSourceProperties里面

自动配置原理：

org.springframework.boot.autoconfigure.jdbc：

1. 参考DataSourceConfiguration：根据配置创建数据源，可以看到，默认是使用apache的，如果配了其他的（HikariDataSource)就配置其他的数据源。用`spring.datasource.type`指定自定义的数据源类型。

2. SpringBoot默认可以支持：

   - org.apache.tomcat.jdbc.pool.DataSource：Tomcat的
   - com.zaxxer.hikari.HikariDataSource
   - org.apache.commons.dbcp.BasicDataSource
   - org.apache.commons.dbcp2.BasicDataSource

3. 自定义数据源类型：

   ```java
   /**
    * Generic DataSource configuration.
    */
   @Configuration
   @ConditionalOnMissingBean(DataSource.class)
   @ConditionalOnProperty(name = "spring.datasource.type")
   static class Generic {
   
       @Bean
       public DataSource dataSource(DataSourceProperties properties) {
           // 使用DataSourceBuilder来创建数据源，利用反射创建相应type的数据源，并且绑定相关属性
           return properties.initializeDataSourceBuilder().build();
       }
   
   }
   ```

4. DataSourceAutoConfiguration里有一个DataSourceInitializer，是一个ApplicationListener

   作用：

   1. ```java
      /**
       * Bean to handle {@link DataSource} initialization by running {@literal schema-*.sql} on
       * {@link PostConstruct} and {@literal data-*.sql} SQL scripts on a
       * {@link DataSourceInitializedEvent}.
       */
      ```

      可以帮我们运行schema-\*.sql文件以及data-\*.sql文件

   2. runSchemaScripts();拿到数据源之后做的操作：运行建表语句，把建表的sql 放在指定位置就可以运行了。

   3. runDataScripts();运行插入数据的sql语句；

   4. 默认只需要将文件命名为：`schema-*.sql`和`data-*.sql`

      - 默认规则：schema.sql，schema-all.sql；

      - 可以通过修改配置文件来指定schema的文件名（是个list类型）

        ```yaml
        spring:
          datasource:
            username: root
            password: root
            url: jdbc:mysql://IP:12389/jdbc?characterEncoding=utf8&useSSL=false
            driver-class-name: com.mysql.jdbc.Driver
            schema:
              - classpath:department.sql
              - classpath:employee.sql
        ```

   5. 操作数据库：JdbcTemplateAutoConfiguration：自动配置了JdbcTemplate操作数据库

      ```java
      @Controller
      public class HelloController {
      
          @Autowired
          private JdbcTemplate template;
      
          @ResponseBody
          @GetMapping("/query")
          public Map<String,Object> map(){
              List<Map<String, Object>> list = template.queryForList("SELECT * FROM department");
              return list.get(0);
          }
          
      }
      ```

      

#### 整合druid数据源

引入maven依赖：

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.1.8</version>
</dependency>
```

然后切换到druid数据源：

修改application配置文件:

```yaml
spring.datasource.type: com.alibaba.druid.pool.DruidDataSource
```

测试是否可以拿到druid连接：

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringBoot06DataJdbcApplicationTests {

    @Autowired
    DataSource dataSource;

    @Test
    public void contextLoads() throws SQLException {
        System.out.println(dataSource);
        System.out.println(dataSource.getClass());
        Connection conn = dataSource.getConnection();
        System.out.println(conn);
    }

}
```

输出：

```
{
	CreateTime:"2019-07-18 11:03:29",
	ActiveCount:0,
	PoolingCount:0,
	CreateCount:0,
	DestroyCount:0,
	CloseCount:0,
	ConnectCount:0,
	Connections:[
	]
}
class com.alibaba.druid.pool.DruidDataSource
2019-07-18 11:03:31.350  INFO 3976 --- [           main] com.alibaba.druid.pool.DruidDataSource   : {dataSource-1} inited
com.mysql.jdbc.JDBC4Connection@682bd3c4
```

可以看到class已经变成druid了



配置Druid相关信息（参数、监控器等）：

application.yaml

```yaml
spring:
  datasource:
    username: root
    password: root
    url: jdbc:mysql://tomxwd.top:12389/jdbc?characterEncoding=utf8&useSSL=false
    driver-class-name: com.mysql.jdbc.Driver
    type: com.alibaba.druid.pool.DruidDataSource
    # 下面是druid配置
    initial-size: 5
    min-idle: 5
    max-active: 20
    max-wait: 60000
    time-between-eviction-runs-millis: 60000
    min-evictable-idle-time-millis: 300000
    validation-query: SELECT 1 FROM DUAL
    test-while-idle: true
    test-on-borrow: false
    test-on-return: false
    use-disposable-connection-facade: true
    # 配置监控统计拦截的filters，去掉后监控页面sql无法统计，‘wall’用于防火墙
    filters: stat,wall,log4j
    maxPoolPreparedStatementPerConnectionSize: 20
    useGlobalDataSourceStat: true
    connectionProperties: druid.stat.mergeSql=true;druid.stat.slowSqlMillis=500
```

执行debug看DataSource的值，发现并没有起作用，原因是属性文件并没有对应起来，底层是通过反射来配置的，而spring并没有给druid适配，那么怎么解决呢，我们需要自己来配置数据源。

```java
@Configuration
public class DruidConfig {

    @ConfigurationProperties(prefix = "spring.datasource")
    @Bean
    public DataSource dataSource(){
        return new DruidDataSource();
    }
}
```

打上`@ConfigurationProperties(prefix = "spring.datasource")`，表示这个bean的各个属性会去配置文件里找对应值。

再次debug会发现起作用了。



配置Druid的监控，即是之前讲过的**注册Servlet**操作，**监控Filter**也是同理：

**StatViewServlet和WebStatFilter**

初始化参数用map来传入到bean的InitParamter中即可起作用。

```java
// 配置Druid的监控
// 1.配置一个管理后台的Servlet
@Bean
public ServletRegistrationBean statViewServlet(){
    ServletRegistrationBean servlet = new ServletRegistrationBean(new StatViewServlet(), "/druid/*");
    Map<String,String> initParams = new HashMap<>();
    initParams.put("loginUsername","admin");
    initParams.put("loginPassword","admin");
    initParams.put("allow","");// 不写或者为null 默认就允许所有访问
    initParams.put("deny","172.16.51.68");
    servlet.setInitParameters(initParams);
    return servlet;
}

// 2.配置一个监控的filter
@Bean
public FilterRegistrationBean webStatFilter(){
    FilterRegistrationBean filter = new FilterRegistrationBean();
    filter.setFilter(new WebStatFilter());

    Map<String,String> map = new HashMap<>();
    map.put("exclusions","*.js,*.css,/druid/*");

    filter.setInitParameters(map);
    filter.setUrlPatterns(Arrays.asList("/*"));
    return filter;
}
```

此时访问`localhost:8080/druid`即可进入监控台。



## 整合MyBatis

![1563433205539](C:\Users\weidou.xie\Desktop\笔记\图\1563433205539.png)

首先要引入Mybatis-starter的依赖

```xml
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>1.3.4</version>
</dependency>
```

![1563433575306](C:\Users\weidou.xie\Desktop\笔记\图\1563433575306.png)



引入Druid数据源并配置：

Druid依赖：

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.1.8</version>
</dependency>
```

application.yaml：

```yaml
spring:
  datasource:
    username: root
    password: root
    url: jdbc:mysql://tomxwd.top:12389/mybatis?characterEncoding=utf8&useSSL=false
    driver-class-name: com.mysql.jdbc.Driver
    type: com.alibaba.druid.pool.DruidDataSource
    # 下面是druid配置
    initial-size: 5
    min-idle: 5
    max-active: 20
    max-wait: 60000
    time-between-eviction-runs-millis: 60000
    min-evictable-idle-time-millis: 300000
    validation-query: SELECT 1 FROM DUAL
    test-while-idle: true
    test-on-borrow: false
    test-on-return: false
    use-disposable-connection-facade: true
    # 配置监控统计拦截的filters，去掉后监控页面sql无法统计，‘wall’用于防火墙
    filters: stat,wall,log4j
    maxPoolPreparedStatementPerConnectionSize: 20
    useGlobalDataSourceStat: true
    connectionProperties: druid.stat.mergeSql=true;druid.stat.slowSqlMillis=500
```

Druid配置（参数和监控等）

```java
@Configuration
public class DruidConfig {

    @ConfigurationProperties(prefix = "spring.datasource")
    @Bean
    public DataSource dataSource(){
        return new DruidDataSource();
    }

    // 配置Druid的监控
    // 1.配置一个管理后台的Servlet
    @Bean
    public ServletRegistrationBean statViewServlet(){
        ServletRegistrationBean servlet = new ServletRegistrationBean(new StatViewServlet(), "/druid/*");
        Map<String,String> initParams = new HashMap<>();
        initParams.put("loginUsername","admin");
        initParams.put("loginPassword","admin");
        initParams.put("allow","");// 不写或者为null 默认就允许所有访问
        initParams.put("deny","172.16.51.68");
        servlet.setInitParameters(initParams);
        return servlet;
    }

    // 2.配置一个监控的filter
    @Bean
    public FilterRegistrationBean webStatFilter(){
        FilterRegistrationBean filter = new FilterRegistrationBean();
        filter.setFilter(new WebStatFilter());

        Map<String,String> map = new HashMap<>();
        map.put("exclusions","*.js,*.css,/druid/*");

        filter.setInitParameters(map);
        filter.setUrlPatterns(Arrays.asList("/*"));
        return filter;
    }


}
```



给数据库建表：

```sql
CREATE TABLE `employee` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `lastName` varchar(255) COLLATE utf8mb4_unicode_ci DEFAULT NULL,
  `email` varchar(255) COLLATE utf8mb4_unicode_ci DEFAULT NULL,
  `gender` int(11) DEFAULT NULL,
  `d_id` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

CREATE TABLE `department` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `departmentName` varchar(255) COLLATE utf8mb4_unicode_ci DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

两个表：部门表和员工表；



建javaBean：

Department：

```java
public class Department {

    private Integer id;
    private String departmentName;

    @Override
    public String toString() {
        return "Department{" +
                "id=" + id +
                ", departmentName='" + departmentName + '\'' +
                '}';
    }

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getDepartmentName() {
        return departmentName;
    }

    public void setDepartmentName(String departmentName) {
        this.departmentName = departmentName;
    }
}
```

Employee：

```java
public class Employee {

    private Integer id;
    private String lastName;
    private String email;
    private Integer gender;

    @Override
    public String toString() {
        return "Employee{" +
                "id=" + id +
                ", lastName='" + lastName + '\'' +
                ", email='" + email + '\'' +
                ", gender=" + gender +
                ", dId=" + dId +
                '}';
    }

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getLastName() {
        return lastName;
    }

    public void setLastName(String lastName) {
        this.lastName = lastName;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    public Integer getGender() {
        return gender;
    }

    public void setGender(Integer gender) {
        this.gender = gender;
    }

    public Integer getdId() {
        return dId;
    }

    public void setdId(Integer dId) {
        this.dId = dId;
    }

    private Integer dId;

}
```



接下来就是mybatis整合，分为注解版和和配置版两个部分：



注解版：

如果要在插入的时候返回自增id需要加@Options属性：`@Options(useGeneratedKeys = true,keyProperty = "id",keyColumn = "id")`

```java
// 指定这是一个操作数据库的Mapper
@Mapper
public interface DepartmentMapper {

    @Select("SELECT * FROM department WHERE id = #{id}")
    Department getDeptById(Integer id);

    @Delete("DELETE FROM department WHERE id=#{id}")
    int deleteDeptById(Integer id);
    @Options(useGeneratedKeys = true,keyProperty = "id",keyColumn = "id")
    @Insert({"INSERT INTO department (departmentName) values (#{departmentName})"})
    int insertDept(Department dept);

    @Update("UPDATE department set departmentName=#{departmentName} where id=#{id}")
    int updateDept(Department dept);

}
```

随便写个Controller进行注入mapper测试，发现都可以的，为什么不用进行相关配置？因为MybatisAutoConfiguration.java文件。

这个MybatisAutoConfiguration类在容器中给我们配好了SqlSessionFactory等。



思考一个问题：如果数据库字段改为department_name会怎么样？测试一下：

会报错的。

所以我们要开启驼峰命名法，怎么做：

发现ConfigurationCustomize，也是可以来自己定制的；

写一个mybatis配置文件类：

```java
@org.springframework.context.annotation.Configuration
public class MyBatisConfig {

    @Bean
    public ConfigurationCustomizer configurationCustomizer(){
        ConfigurationCustomizer customizer = new ConfigurationCustomizer() {
            @Override
            public void customize(Configuration configuration) {
                // 开启驼峰命名规则
                configuration.setMapUnderscoreToCamelCase(true);
            }
        };
        return customizer;
    }

}
```

这个时候就开启了驼峰命名，重启项目，发现可以了。



那么，如果每一个Mapper都要自己去加`@Mapper`注解，显得太麻烦，而不加又不行，怎么办呢。我们可以用`@MapperScan`注解来标明Mapper扫描包。可以在任意一个配置类上加这个注解，但是最好在SpringBoot启动类或者mybatis配置类上加吧，规范。

```java
@org.springframework.context.annotation.Configuration
@MapperScan(basePackages = {"top.tomxwd.mapper"})
public class MyBatisConfig {

    @Bean
    public ConfigurationCustomizer configurationCustomizer(){
        ConfigurationCustomizer customizer = new ConfigurationCustomizer() {
            @Override
            public void customize(Configuration configuration) {
                // 开启驼峰命名规则
                configuration.setMapUnderscoreToCamelCase(true);
            }
        };
        return customizer;
    }

}
```





配置版：

创建一个新的Mapper

**不管是配置版的还是注解版的，都需要用@Mapper或者@MapperScan将接口扫描装配到容器中;**

```java
// @Mapper或者@MapperScan将接口扫描装配到容器中
public interface EmployeeMapper {

    Employee getEmpById(Integer id);
    
    Integer insertEmp(Employee employee);

}
```

创建一个目录以及Mapper.xml文件。

而详细的配置到官网查看；

[mybatis-github地址](https://github.com/mybatis/mybatis-3)

[mybatis官方文档](http://www.mybatis.org/mybatis-3/)

getting start里面开头有个全局配置文件：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
  <environments default="development">
    <environment id="development">
      <transactionManager type="JDBC"/>
      <dataSource type="POOLED">
        <property name="driver" value="${driver}"/>
        <property name="url" value="${url}"/>
        <property name="username" value="${username}"/>
        <property name="password" value="${password}"/>
      </dataSource>
    </environment>
  </environments>
  <mappers>
    <mapper resource="org/mybatis/example/BlogMapper.xml"/>
  </mappers>
</configuration>
```

内容不要，仅仅是复制过来看看。

翻到下面有sql映射文件的示例：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="org.mybatis.example.BlogMapper">
  <select id="selectBlog" resultType="Blog">
    select * from Blog where id = #{id}
  </select>
</mapper>
```

那么，修改命名空间以及内容：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="top.tomxwd.mapper.EmployeeMapper">

    <!--
        Employee getEmpById(Integer id);
        Integer insertEmp(Employee employee);
    -->
    <select id="getEmpById" resultType="top.tomxwd.bean.Employee">
        SELECT * FROM employee WHERE id=#{id}
    </select>

    <insert id="insertEmp">
        INSERT INTO employee(lastName,email,gender,d_id) VALUE (#{lastName},#{email},#{gender},#{dId})
    </insert>
    
</mapper>
```

这个时候还需要注意，mapper.xml并没有被扫描到，所以需要在application.yaml配置文件加上以下内容：

```yaml
mybatis:
  config-location: classpath:mybatis/mybatis-config.xml
  mapper-locations: classpath:mybatis/mapper/*.xml
```

随便编写一个Controller测试一下，发现配置类做的驼峰命名规则无效了，因为配置文件里指定了mybatis-config.xml文件，优先级高，所以屏蔽了配置类的作用，要么就不要指定这个xml文件，要么就在xml文件里配置驼峰命名规则。

这时候再测试一下注解版的可以用不，发现可以，这样两种配置都可以混合使用了。



## 整合JPA

![1563440124653](C:\Users\weidou.xie\Desktop\笔记\图\1563440124653.png)

介绍一下SpringData



![1563440197016](C:\Users\weidou.xie\Desktop\笔记\图\1563440197016.png)

// TODO

![1563440241198](C:\Users\weidou.xie\Desktop\笔记\图\1563440241198.png)

// TODO

![1563440290644](C:\Users\weidou.xie\Desktop\笔记\图\1563440290644.png)



// TODO

JPA规范：JSR 317  JPA全名是Java Persistence API 顾名思义java持久层的接口规范



![1563440094464](C:\Users\weidou.xie\Desktop\笔记\图\1563440094464.png)

引入依赖pom.xml：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <scope>runtime</scope>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>

```

查看依赖图，看到JPA其实底层就是用hibernate实现的。



配置数据源：

```yaml
spring:
  datasource:
    url: jdbc:mysql://tomxwd.top:12389/jpa
    username: root
    password: root
    driver-class-name: com.mysql.jdbc.Driver
```

JPA:ORM(Object Relational Mapping);

1. 编写一个实体类（entity）和数据表进行映射；并且配置好映射关系。

   ```java
   // 使用JPA注解配置映射关系
   @Entity //@Entity告诉JPA这是一个实体类（和数据表映射的类）
   @Table(name="tbl_user") //@Table来指定和哪个数据表对应；如果省略，默认表名就是user
   public class User {
   
       @Id// @Id表示这是一个主键
       @GeneratedValue(strategy = GenerationType.IDENTITY) // 自增主键
       private Integer id;
   
       @Column(name="last_name",length = 50) // 这是和数据表对应的一个列,省略默认列名就是属性名
       private String lastName;
   
       @Column(name="email")
       private String email;
   ```

2. 编写一个Dao接口来操作实体类对应的数据表（Repository）

   ```java
   // 继承JpaRepository来完成对数据库的操作
   public interface UserRepository extends JpaRepository<User,Integer> {
   
   }
   ```

3. 修改主配置文件application.yaml

   ```yaml
   spring:
     datasource:
       url: jdbc:mysql://ip:12389/jpa?characterEncoding=utf8&useSSL=false
       username: root
       password: root
       driver-class-name: com.mysql.jdbc.Driver
     jpa:
       hibernate:
   #      更新或者创建数据表结构
         ddl-auto: update
   #      每次增删改查的时候输出sql语句
       show-sql: true
   ```

   可以根据实体类自动生成表结构，以及输出sql语句。

4. 运行项目，会发现数据库创建了tbl_user表。

5. 写一个controller来测试一下：

   ```java
   @RestController
   public class UserController {
   
       @Autowired
       private UserRepository repository;
   
       @GetMapping("/user/{id}")
       public User getUser(@PathVariable("id") Integer id){
           User user = repository.findOne(id);
           return user;
       }
   
       @GetMapping("/user")
       public User insertUser(User user){
           User save = repository.save(user);
           return save;
       }
   
   
   }
   ```

   先insert再select，发现控制台会打印sql



# SpringBoot启动配置原理

1. 启动原理
2. 运行流程
3. 自动配置原理



几个重要的事件回调机制

配置在META-INF/spring.factories中：

ApplicationContextInitializer

SpringApplicationRunListener



只需要加在IOC容器中：

ApplicationRunner

CommandLineRunner



![1563498550479](C:\Users\weidou.xie\Desktop\笔记\图\1563498550479.png)

先创建一个简单的SpringBoot-web工程

断点打在启动类的main方法上进行debug

启动流程：

1. 创建SpringApplication对象

   ```java
   initialize(sources);
   
   private void initialize(Object[] sources) {
       // 保存主配置类
       if (sources != null && sources.length > 0) {
           this.sources.addAll(Arrays.asList(sources));
       }
       // 判断当前应用是否为web应用
       this.webEnvironment = deduceWebEnvironment();
       // 从类路径下找到META-INF/spring.factories配置的所有ApplicationContextInitializer；然后保存起来。
       setInitializers((Collection) getSpringFactoriesInstances(
           ApplicationContextInitializer.class));
       // 还是从类路径下找到META-INF/spring.factories配置的所有ApplicationListener
       setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
       // 从多个配置类中找到有main方法的主配置类
       this.mainApplicationClass = deduceMainApplicationClass();
   }
   ```

   把这些initializers都保存起来

   ![1563500522617](C:\Users\weidou.xie\Desktop\笔记\图\1563500522617.png)

   把这些listener也保存起来

   ![1563501112980](C:\Users\weidou.xie\Desktop\笔记\图\1563501112980.png)

2. 运行run方法

   ```java
   public ConfigurableApplicationContext run(String... args) {
       // 停止的监听
       StopWatch stopWatch = new StopWatch();
       stopWatch.start();
       // 声明一个ioc容器，指向null
       ConfigurableApplicationContext context = null;
       FailureAnalyzers analyzers = null;
       // 做awt有关的
       configureHeadlessProperty();
       
       // 获取SpringApplicationRunListeners，是从类路径下META-INF/spring.factories
       SpringApplicationRunListeners listeners = getRunListeners(args);
       // 回调所有的SpringApplicationRunListeners的starting()方法
       listeners.starting();
       try {
           // 封装命令行参数
           ApplicationArguments applicationArguments = new DefaultApplicationArguments(
               args);
           // 准备环境 
           	// 1. 创建环境完成后，会回调所有SpringApplicationRunListeners的environmentPrepared()方法：表示环境准备完成
           	// 2. 如果是web环境还会转成web环境
           ConfigurableEnvironment environment = prepareEnvironment(listeners,applicationArguments);
                
           // 打印Banner图标
           Banner printedBanner = printBanner(environment);
           
           // 创建ioc容器ApplicationContext
           	// 1. 根据你的环境，决定创建的web环境与否，直接是利用全限定名然后用BeanUtil进行反射创建对象return回去。
           context = createApplicationContext();
           // 主要是用来发生异常的时候做异常分析报告的
           analyzers = new FailureAnalyzers(context);
           
           // 准备上下文环境
           	// 将environment保存到ioc容器当中；
           	// 而且调用applyInitializers();回调之前保存的所有的ApplicationContextInitializer的initialize()方法
           	// 还要回调所有的SpringApplicationRunListener的contextPrepared()方法；
           	// prepareContext运行完成后，还要回调所有的SpringApplicationRunListener的contextLoaded()方法
           prepareContext(context, environment, listeners, applicationArguments,
                          printedBanner);
           
           // 刷新容器：IOC容器初始化；重点关注refresh()方法的finishBeanFactoryInitialization() 创建所有的单例Bean和容器中所有的组件
           // 如果是web应用还会创建嵌入式的tomcat等
           // **扫描，创建，加载所有组件的地方**(配置类，组件，自动配置)
           refreshContext(context);
           
           // 从IOC容器中获取所有的ApplicationRunner和CommandLineRunner进行回调
           	// ApplicationRunner先回调，然后CommandLineRunner回调
           afterRefresh(context, applicationArguments);
           
           // 所有的SpringApplicationRunListener回调finished()方法
           listeners.finished(context, null);
           
           // 监听结束
           stopWatch.stop();
           if (this.logStartupInfo) {
               new StartupInfoLogger(this.mainApplicationClass)
                   .logStarted(getApplicationLog(), stopWatch);
           }
           
           //整个SpringBoot应用启动完成以后返回启动的IOC容器
           return context;
       }
       catch (Throwable ex) {
           handleRunFailure(context, listeners, analyzers, ex);
           throw new IllegalStateException(ex);
       }
   }
   ```

   找到的Listener：

   ![1563502287477](C:\Users\weidou.xie\Desktop\笔记\图\1563502287477.png)

   refresh方法：

   ```java
   @Override
   	public void refresh() throws BeansException, IllegalStateException {
   		synchronized (this.startupShutdownMonitor) {
   			// Prepare this context for refreshing.
   			prepareRefresh();
   
   			// Tell the subclass to refresh the internal bean factory.
   			ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
   
   			// Prepare the bean factory for use in this context.
   			prepareBeanFactory(beanFactory);
   
   			try {
   				// Allows post-processing of the bean factory in context subclasses.
   				postProcessBeanFactory(beanFactory);
   
   				// Invoke factory processors registered as beans in the context.
   				invokeBeanFactoryPostProcessors(beanFactory);
   
   				// Register bean processors that intercept bean creation.
   				registerBeanPostProcessors(beanFactory);
   
   				// Initialize message source for this context.
   				initMessageSource();
   
   				// Initialize event multicaster for this context.
   				initApplicationEventMulticaster();
   
   				// Initialize other special beans in specific context subclasses.
   				onRefresh();
   
   				// Check for listener beans and register them.
   				registerListeners();
   
   				// Instantiate all remaining (non-lazy-init) singletons.
   				finishBeanFactoryInitialization(beanFactory);
   
   				// Last step: publish corresponding event.
   				finishRefresh();
   			}
   
   			catch (BeansException ex) {
   				if (logger.isWarnEnabled()) {
   					logger.warn("Exception encountered during context initialization - " +
   							"cancelling refresh attempt: " + ex);
   				}
   
   				// Destroy already created singletons to avoid dangling resources.
   				destroyBeans();
   
   				// Reset 'active' flag.
   				cancelRefresh(ex);
   
   				// Propagate exception to caller.
   				throw ex;
   			}
   
   			finally {
   				// Reset common introspection caches in Spring's core, since we
   				// might not ever need metadata for singleton beans anymore...
   				resetCommonCaches();
   			}
   		}
   	}
   ```

3. 事件监听机制

   配置在META-INF/spring.factories中：

   **ApplicationContextInitializer**

   **SpringApplicationRunListener**

   

   只需要加在IOC容器中：

   **ApplicationRunner**

   **CommandLineRunner**

   

   首先创建4个类分别实现上述几个接口，并进行实现，注意**SpringApplicationRunListener**需要构造器，参考别的实现即可。

   

   ApplicationContextInitializer:

   ```java
   public class HelloApplicationContextInitializer implements ApplicationContextInitializer<ConfigurableApplicationContext> {
   
   
       @Override
       public void initialize(ConfigurableApplicationContext applicationContext) {
           System.out.println("HelloApplicationContextInitializer.initialize");
           System.out.println("初始化容器"+applicationContext);
       }
   }
   ```

   

   SpringApplicationRunListener:

   ```java
   public class HelloSpringApplicationRunListener implements SpringApplicationRunListener {
   
       public HelloSpringApplicationRunListener(SpringApplication application,String[] args) {
           System.out.println("HelloSpringApplicationRunListener.HelloSpringApplicationRunListener");
       }
   
       @Override
       public void starting() {
           System.out.println("HelloSpringApplicationRunListener.starting");
       }
   
       @Override
       public void environmentPrepared(ConfigurableEnvironment environment) {
           Object o = environment.getSystemProperties().get("os.name");
           System.out.println("HelloSpringApplicationRunListener.environmentPrepared"+o);
       }
   
       @Override
       public void contextPrepared(ConfigurableApplicationContext context) {
           System.out.println("HelloSpringApplicationRunListener.contextPrepared");
       }
   
       @Override
       public void contextLoaded(ConfigurableApplicationContext context) {
           System.out.println("HelloSpringApplicationRunListener.contextLoaded");
       }
   
       @Override
       public void finished(ConfigurableApplicationContext context, Throwable exception) {
           System.out.println("HelloSpringApplicationRunListener.finished");
       }
   }
   ```

   

   ApplicationRunner:

   ```java
   @Component
   public class HelloApplicationRunner implements ApplicationRunner {
   
       @Override
       public void run(ApplicationArguments args) throws Exception {
           System.out.println("HelloApplicationRunner.run");
       }
   }
   ```

   

   CommandLineRunner:

   ```java
   @Component
   public class HelloCommandLineRunner implements CommandLineRunner {
   
       @Override
       public void run(String... args) throws Exception {
           System.out.println("HelloCommandLineRunner.run"+ Arrays.asList(args));
       }
   }
   ```

   

   接着在类路径下创建META-INF文件夹，里面编写一个spring.factories文件，因为**ApplicationContextInitializer和SpringApplicationRunListener**需要这样配置：

   ```properties
   org.springframework.context.ApplicationContextInitializer=\
     top.tomxwd.listener.HelloApplicationContextInitializer
   org.springframework.boot.SpringApplicationRunListener=\
     top.tomxwd.listener.HelloSpringApplicationRunListener
   ```

   而**ApplicationRunner和CommandLineRunner**则需要在类上打个`@Component`注解即可被扫描到。

   这时候启动应用，就会看到输出结果。



# SpringBoot自定义starters

> starters原理、自定义starters

![1563516815526](C:\Users\weidou.xie\Desktop\笔记\图\1563516815526.png)

starter：

1. 这个场景需要用到的依赖是什么？

2. 如何编写自动配置

   ```java
   @Configuration //指定这个类是一个配置类
   @ConditionalOnXxx //在指定条件成立的情况下，自动配置类生效
   @AutoConfigureOrder //自动配置类的顺序
   @AutoConfigureAfter //在哪个自动配置类之后生效
   @Bean // 给容器中添加组件
   
   @ConfigurationProperties//结合相关的XxxProperties类来绑定相关的配置
   @EnableCOnfigurationProperties//让XxxProperties类生效并加入到容器中，就可以自动装配了
   
   // 自动配置类要能被加载，需要把这些自动配置类加入到/META-INF/spring.factories
   ```

3. 模式

   ![1563517036591](C:\Users\weidou.xie\Desktop\笔记\图\1563517036591.png)

   启动器只用来做依赖导入，然后专门写一个自动配置模块；

   启动器来依赖自动配置模块，别人只需要引入启动器（starter）；

   自定义的启动器命名：类似mybatis-spring-boot-starter；

4. 定义自己的starter

   首先创建两个项目，一个作为启动器（starter），里面只需要引入自动配置模块即可，这里先创建一个Maven项目然后依赖自动配置模块；另一个项目则依赖SpringBoot启动器，然后编写相关自动配置需要的文件。

   启动器项目：

   命名为：tomxwd-spring-boot-starter

   ```xml
   <!-- 启动器 -->
   <dependencies>
       <!-- 引入自动配置模块 -->
       <dependency>
           <groupId>top.tomxwd.starter</groupId>
           <artifactId>tomxwd-spring-boot-starter-autoconfigurer</artifactId>
           <version>0.0.1-SNAPSHOT</version>
       </dependency>
   </dependencies>
   ```

   修改项目的pom文件即可。

   自动配置模块：

   命名为：tomxwd-spring-boot-starter-autoconfigurer

   pom文件：

   ```xml
   <dependencies>
       <!-- 引入SpringBoot-starter；所有starter的基本配置 -->
       <dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter</artifactId>
       </dependency>
   </dependencies>
   ```

   编写一个组件(HelloService)：

   ```java
   public class HelloService {
   
       HelloProperties helloProperties;
   
       public HelloProperties getHelloProperties() {
           return helloProperties;
       }
   
       public void setHelloProperties(HelloProperties helloProperties) {
           this.helloProperties = helloProperties;
       }
   
       public String sayHelloTomxwd(String name){
           return helloProperties.getPrefix()+"-"+name+helloProperties.getSuffix();
       }
   
   }
   ```

   编写自动配置类（HelloServiceAutoConfiguration）：

   ```java
   @Configuration // 标明这是一个配置类
   @ConditionalOnWebApplication//WEB应用的情况下，才生效
   @EnableConfigurationProperties({HelloProperties.class})//导入配置文件
   public class HelloServiceAutoConfiguration {
   
       @Autowired
       HelloProperties helloProperties;
   
       @Bean
       public HelloService helloService(){
           HelloService helloService = new HelloService();
           helloService.setHelloProperties(helloProperties);
           return helloService;
       }
   
   }
   ```

   编写属性类（HelloProperties）：

   ```java
   @ConfigurationProperties(prefix = "tomxwd.hello")
   public class HelloProperties {
   
       private String prefix;
       private String suffix;
   
       public String getPrefix() {
           return prefix;
       }
   
       public void setPrefix(String prefix) {
           this.prefix = prefix;
       }
   
       public String getSuffix() {
           return suffix;
       }
   
       public void setSuffix(String suffix) {
           this.suffix = suffix;
       }
   }
   ```

   然后再在类路径下创建META-INF文件夹，在该文件夹中创建一个spring.factories文件，然后把自动配置类交给EnableAutoConfiguration：

   ```properties
   org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
     top.tomxwd.starter.HelloServiceAutoConfiguration
   ```

   然后进行打包，先打包自动配置模块，因为启动器依赖于这个自动配置模块。

   打包完成后，进行测试，创建一个SpringBoot的Web 工程，在pom文件中进行依赖启动器：

   ```xml
   <!-- 引入我们自定义的starter -->
   <dependency>
       <groupId>top.tomxwd.starter</groupId>
       <artifactId>tomxwd-springboot-starter</artifactId>
       <version>1.0-SNAPSHOT</version>
   </dependency>
   ```

   编写一个Controller进行测试：

   ```java
   @RestController
   public class HelloController {
   
       @Autowired
       private HelloService service;
   
       @GetMapping("/say")
       public String say(){
           String say = service.sayHelloTomxwd("哈哈");
           return say;
       }
   
   }
   ```

   在application.properties中进行配置：

   ```properties
   tomxwd.hello.prefix=TOMXWD
   tomxwd.hello.suffix=DONE
   ```

   启动项目并进行访问，得到结果；

   

   



