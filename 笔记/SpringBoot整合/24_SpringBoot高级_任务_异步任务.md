---
2019-08-14 08:56:37

---





springBoot版本2.1.7

Controller:

```java
@RestController
public class AsyncController {

    @Autowired
    private AsyncService service;

    @GetMapping("/hello")
    public String hello(){
        service.helle();
        return "success";
    }


}
```



Service:

```java
@Service
public class AsyncService {

    // 告诉spring这是一个异步方法，spring会自己开一个线程池来进行调用
    @Async
    public void helle(){
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("数据处理中。。。");
    }

}
```



SpringBoot启动类：

```java
@SpringBootApplication
// 开启异步
@EnableAsync
public class Springboot04TaskApplication {

    public static void main(String[] args) {
        SpringApplication.run(Springboot04TaskApplication.class, args);
    }

}
```

