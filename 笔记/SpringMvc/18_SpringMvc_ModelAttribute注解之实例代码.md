---
2019-07-25 11:15:52

---



1. ```jsp
   <hr>
   <%--模拟修改操作
           1. 原始数据：1，Tom，123456,tom@qq.com,12
       2. 要求密码不能被修改
           3. 表单回显，模拟操作直接在表单填写对应的属性值
       --%>
   <form action="/modelAtt" method="post">
       <input type="hidden" name="id" value="1">
       <input type="text" name="username" value="tom">
       <br>
       <input type="text" name="email" value="tom@qq.com">
       <br>
       <input type="text" name="age" value="12">
       <br>
       <input type="submit" value="submit">
   </form>
   
   <hr>
   ```

2. ```java
   /**
    * 有ModelAttribute注解标记的方法，会在每个目标方法执行之前被SpringMvc调用
    * @param id
    * @param map
    */
   @ModelAttribute
   public void getUser(@RequestParam(value = "id",required = false)Integer id,Map<String,Object> map){
       if(id!=null){
           //修改操作
           // 模拟从数据库中取出对象
           User user = new User(1, "Tom", "123456", "tom@qq.com", 22);
           map.put("user",user);
           System.out.println("数据库中获取一个User "+user);
       }
   }
   
   @RequestMapping(value = "/modelAtt",method = RequestMethod.POST)
   public String modelAtt(User user){
       System.out.println("修改"+user);
       return SUCCESS;
   }
   ```

