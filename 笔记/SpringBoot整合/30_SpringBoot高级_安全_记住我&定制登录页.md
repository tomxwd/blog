---
2019-08-14 17:47:05

---

 

记住我功能：

在配置文件中加：

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    // 开启记住我功能
    http.rememberMe();
}
```

原理是SpringSecurity在cookie中添加一个叫remember-me的Cookie，默认是保存两周；注销的时候也会跟着被删除。