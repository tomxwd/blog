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



Docker核心概念

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



p=55

