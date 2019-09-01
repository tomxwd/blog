---
2019-07-24 16:56:51
---



![1563958631972](图/1563958631972.png)

```java
@RequestMapping("antPath/*/abc")
public String testAntPath(){
    System.out.println("RequestMethodController.testAntPath");
    return SUCCESS;
}
```

