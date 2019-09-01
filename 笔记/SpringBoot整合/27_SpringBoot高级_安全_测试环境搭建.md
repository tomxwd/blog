---
2019-08-14 14:28:14
---



![1565764100587](../数据结构/数据结构图解/1565764100587.png)

![1565764111259](../数据结构/数据结构图解/1565764111259.png)

Controller层：

```java
@Controller
public class KungfuController {

    private final String PREFIX="pages/";

    @GetMapping("/")
    public String index(){
        return "welcome";
    }

    @GetMapping("/userLogin")
    public String userLogin(){
        return PREFIX+"login";
    }

    @GetMapping("/level1/{path}")
    public String level1(@PathVariable("path")String path){
        return PREFIX+"level1/"+path;
    }

    @GetMapping("/level2/{path}")
    public String level2(@PathVariable("path")String path){
        return PREFIX+"level2/"+path;
    }

    @GetMapping("/level3/{path}")
    public String level3(@PathVariable("path")String path){
        return PREFIX+"level3/"+path;
    }

}
```

欢迎页面：

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>欢迎页面</title>
</head>
<body>
<h1><a th:href="@{/userLogin}">请登录</a></h1>
<hr>
<h2>level1</h2>
<h3><a th:href="@{/level1/1}">level1-1</a></h3>
<h3><a th:href="@{/level1/2}">level1-2</a></h3>
<h3><a th:href="@{/level1/3}">level1-3</a></h3>
<hr>
<h2>level2</h2>
<h3><a th:href="@{/level2/1}">level2-1</a></h3>
<h3><a th:href="@{/level2/2}">level2-2</a></h3>
<h3><a th:href="@{/level2/3}">level2-3</a></h3>
<hr>
<h2>level3</h2>
<h3><a th:href="@{/level3/1}">level3-1</a></h3>
<h3><a th:href="@{/level3/2}">level3-2</a></h3>
<h3><a th:href="@{/level3/3}">level3-3</a></h3>
</body>
</html>
```

具体页面：

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>level1--1</title>
</head>
<body>
<h1>level1--1</h1>
<h3><a th:href="@{/}">返回</a></h3>
</body>
</html>
```

