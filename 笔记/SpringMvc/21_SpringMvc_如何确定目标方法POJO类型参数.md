---
2019-07-25 14:36:01
---





















SpringMVC确定目标方法POJO类型入参的过程

1. 确定一个key：
   - 若目标方法的POJO类型的参数没有使用@ModelAttribute作为修饰，则key为pojo类名第一个字母小写。
   - 若使用了@ModelAttribute来修饰，则key为@ModelAttribute注解的value属性值。
2. 在implicitModel中存在key对应的对象，若存在，则作为入参传入。
   - 若在@ModelAttribute标记的方法中在Map中保存过，且key和1确定的key一致，则会获取到。
3. 在implicitModel中不存在key对应的对象，则检查当前的Handler是否使用@SessionAttributes注解修饰，若使用了该注解，且@SessionAttributes注解的value属性中包含了key，则会从HttpSession中来获取key所对应的value值，若存在则直接传入到目标方法的入参中，若不存在，则抛出异常。
4. 若Handler没有标识@SessionAttributes注解，或@SessionAttributes注解的value值中不包含key，则会通过反射来创建POJO类型的参数，传入为目标方法的参数。
5. SpringMvc会把key和POJO类型的对象保存到implicitModel中，进而会保存到request中。