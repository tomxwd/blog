---
2019-07-25 11:05:49

---





1. ![1564022691705](../../%E7%AC%94%E8%AE%B0/SpringMvc/%E5%9B%BE/1564022691705.png)

2. ```java
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

3. 注意：该注解要放在类上