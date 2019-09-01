---
2019-07-25 11:03:19

---





1. ![1564022321654](../../%E7%AC%94%E8%AE%B0/SpringMvc/%E5%9B%BE/1564022321654.png)

2. ```java
   /**
    * 目标方法可以添加Map类型（实际上也可以是Model类型或ModelMap类型）的参数
    * @param map
    * @return
    */
   @RequestMapping("/map")
   public String testMap(Map<String,Object> map){
       map.put("names", Arrays.asList("Tom","Jerry","Mike"));
       return SUCCESS;
   }
   ```