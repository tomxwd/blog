---
2019-07-25 10:02:55

---





1. ![1564020185176](图/1564020185176.png)

2. ```java
   @RequestMapping(value = "testPojo",method = RequestMethod.POST)
   public String testPojo(User user){
       System.out.println("RequestMethodController.testPojo User:"+user);
       return SUCCESS;
   }
   ```

3. 

