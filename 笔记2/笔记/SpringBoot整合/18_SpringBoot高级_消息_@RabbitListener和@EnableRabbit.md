---
2019-08-02 11:23:40

---





@EnableRabbit+@RabbitListener，监听消息队列的内容；

主配置类：

```java
@SpringBootApplication
@EnableRabbit
public class Springboot03AmqpApplication {

    public static void main(String[] args) {
        SpringApplication.run(Springboot03AmqpApplication.class, args);
    }

}
```

service类：

```java
@Service
public class BookService {

    @RabbitListener(queues = "atguigu.news")
    public void receive(Book book){
        System.out.println("收到消息"+book);
    }

    @RabbitListener(queues = "atguigu")
    public void receive02(Message message){
        System.out.println("message.getBody() = " + message.getBody());
        System.out.println("message.getMessageProperties() = " + message.getMessageProperties());
    }

}
```