---
2019-07-25 13:51:05

---



```java
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

/**
     * 运行流程：
     * 1. 执行@ModelAttribute 注解修饰的方法：从数据库中取出对象，把对象放入到map中，键为：user
     * 2. SpringMvc从map中取出user对象，并把表单的请求参数赋给该user对象的对应属性
     * 3. SpringMvc把上述对象传入目标方法的参数。
     *
     * 注意：在@ModelAttribute修饰的方法中，放入到Map时的键需要和目标方法的入参类型的第一个字母小写的字符串一致
*/
@RequestMapping(value = "/modelAtt",method = RequestMethod.POST)
public String modelAtt(User user){
    System.out.println("修改"+user);
    return SUCCESS;
}
```



