---
2019-08-02 13:45:54
---







在使用发消息以及接受消息的时候，前提是创建好了exchange和queue的信息以及绑定规则等，再此之前我们是手动去web页面创建的，如果没有这个web页面，我们如何去创建，这时候就要用到AMQPadmin管理组件。



创建exchange：

```java
@Test
public void createExchange(){
    // 创建一个exchange
    amqpAdmin.declareExchange(new DirectExchange("amqpadmin.exchange"));
    System.out.println("创建完成");
}
```

创建queue：

```java
@Test
public void createQueue(){
    // 创建一个queue
    amqpAdmin.declareQueue(new Queue("amqpadmin.queue",true));
    System.out.println("创建完成");
}
```

创建绑定规则：

```java
@Test
public void createBinding(){
    amqpAdmin.declareBinding(new Binding("amqpadmin.queue",Binding.DestinationType.QUEUE,"amqpadmin.exchange","amqp.haha",null));
    System.out.println("创建完成");
}
```

结合rabbit的web页面看结果；