---
2019-07-24 16:37:05
---

![1563957465481](图/1563957465481.png)





```java
/**
* 了解：可以使用params和headers来更加精确映射请求
* params和headers支持简单的表达式。
* @return
*/
@RequestMapping(value = "/paramsAndHeaders",
                params = {"username","age!=10"},
                headers = {"Accept-Language=zh-CN,zh;q=0.9"})
public String testParamsAndHeaders(){
    System.out.println("RequestMethodController.testParamsAndHeaders");
    return SUCCESS;
}
```

