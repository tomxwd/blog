---
2019-07-25 14:11:53
---







源代码分析的流程

1. 调用@ModelAttribute注解修饰的方法。实际上把@ModelAttribute方法中的Map中的数据放在了implicitModel中。

2. 解析请求处理器的目标参数，实际上该目标参数来自于WebDataBinder对象的target属性。

   1. 创建WebDataBinder对象：

      - 确定objectName属性：
        - 若传入的attrName属性值为""，则objcetName为类名第一个字母小写。
        - **注意**：**attrName，若目标方法的POJO属性使用了@ModelAttribute来修饰，则attrName值即为@ModelAttribute的value属性值。**

      - 确定target属性：
        - 在implicitModel中查找attrName对应的属性值：
          - 若存在，ok返回。
          - 若不存在：则验证当前Handler是否使用了@SessionAttributes进行修饰，若使用了，则尝试从Session中获取attrName所对应的属性值，**若session中没有对应的属性值，则抛出异常**。
          - 若Handler没有使用@SessionAttributes进行修饰，或者@SessionAttributes中没有使用value值指定的key和attrName相匹配，则通过反射创建pojo对象。

   2. SpringMvc把表单的请求参数赋给了WebDataBinder的target对应的属性。

   3. SpringMvc会把WebDataBinder的attrName和target给到implicitModel，进而传到request域对象中。

   4. 把WebDataBinder的target作为参数传递给目标方法的入参。