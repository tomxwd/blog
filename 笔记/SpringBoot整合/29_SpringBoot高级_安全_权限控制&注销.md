---
2019-08-14 17:19:15
---







注销

在配置类中加内容，关于注销的功能：

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    // 开启自动配置的注销功能   效果：
    http.logout().logoutSuccessUrl("/");
    // 1. /logout表示用户注销，清空session
    // 2. 注销成功默认会返回 /login?logout页面
    // 3. 可以定制返回页面规则,如返回首页
}
```

在欢迎页面加表单：

```html
<form th:action="@{/logout}" method="post">
    <input type="submit" value="注销">
</form>
```

权限控制：

页面的内容控制：

引入依赖：

```xml
<dependency>
    <groupId>org.thymeleaf.extras</groupId>
    <artifactId>thymeleaf-extras-springsecurity5</artifactId>
</dependency>
```

页面添加命名空间，以及一些sec的用法：

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org"
      xmlns:sec="http://www.thymeleaf.org/extras/spring-security">
<head>
    <meta charset="UTF-8">
    <title>欢迎页面</title>
</head>
<body>
<div sec:authorize="!isAuthenticated()">
    <h1><a th:href="@{/login}">请登录</a></h1>
</div>
<div sec:authorize="isAuthenticated()">
    <h2><span sec:authentication="name"></span>,您好，您的角色有：
        <span sec:authentication="principal.authorities"></span></h2>
    <form th:action="@{/logout}" method="post">
        <input type="submit" value="注销">
    </form>
</div>
<div sec:authorize="hasRole('VIP1')">
<hr>
<h2>level1</h2>
<h3><a th:href="@{/level1/1}">level1-1</a></h3>
<h3><a th:href="@{/level1/2}">level1-2</a></h3>
<h3><a th:href="@{/level1/3}">level1-3</a></h3>
</div>
<hr>
<div sec:authorize="hasRole('VIP2')">
<h2>level2</h2>
<h3><a th:href="@{/level2/1}">level2-1</a></h3>
<h3><a th:href="@{/level2/2}">level2-2</a></h3>
<h3><a th:href="@{/level2/3}">level2-3</a></h3>
<hr>
</div>
<div sec:authorize="hasRole('VIP3')">
<h2>level3</h2>
<h3><a th:href="@{/level3/1}">level3-1</a></h3>
<h3><a th:href="@{/level3/2}">level3-2</a></h3>
<h3><a th:href="@{/level3/3}">level3-3</a></h3>
</div>
</body>
</html>
```

