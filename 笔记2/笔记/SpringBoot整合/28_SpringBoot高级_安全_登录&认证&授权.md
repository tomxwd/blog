---
2019-08-14 14:29:59

---



1. 引入SpringSecurity依赖

   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-security</artifactId>
   </dependency>
   ```

2. 编写SpringSecurity的配置类

   - @EnableWebSecurity

   - extends WebSecurityConfigurerAdapter

     ```java
     @EnableWebSecurity
     public class MySecurityConfig extends WebSecurityConfigurerAdapter {}
     ```

3. 控制请求的访问权限

   - 由于5.0版本改了，默认会有加密，所以实现一个不加密的形式：

     ```java
     public class MyPasswordEncoder implements PasswordEncoder {
     
         @Override
         public String encode(CharSequence charSequence) {
             return charSequence.toString();
         }
     
         @Override
         public boolean matches(CharSequence charSequence, String s) {
             return s.equals(charSequence.toString());
         }
     
     }
     ```

   - 配置类

     ```java
     @EnableWebSecurity
     public class MySecurityConfig extends WebSecurityConfigurerAdapter {
     
         @Override
         protected void configure(HttpSecurity http) throws Exception {
             // 定制请求的授权规则
             http.authorizeRequests().antMatchers("/").permitAll()
                     .antMatchers("/level1/**").hasRole("VIP1")
                     .antMatchers("/level2/**").hasRole("VIP2")
                     .antMatchers("/level3/**").hasRole("VIP3");
             // 开启自动配置的登录功能  效果：如果没有权限就会来到登录页面
             http.formLogin();
             // 1. /login来到登录页
             // 2. 重定向到/login?error表示登录失败
             // 3. 更多详细规则
         }
     
         // 定义认证规则
         @Override
         protected void configure(AuthenticationManagerBuilder auth) throws Exception {
             auth.inMemoryAuthentication()
                     .passwordEncoder(new MyPasswordEncoder())
                     .withUser("zhangsan").password("123456").roles("VIP1","VIP2")
                     .and()
                     .withUser("lisi").password("123456").roles("VIP2","VIP3")
                     .and()
                     .withUser("wangwu").password("123456").roles("VIP1","VIP3");
         }
     
     }
     ```

4. 默认是/login作为登录的url，SpringSecurity自带的，如果错误会重定向到/login?error；

