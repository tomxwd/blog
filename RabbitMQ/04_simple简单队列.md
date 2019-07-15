---
title: 04_simple简单队列
data: 2019-06-13 ‏‎‏‎22:22:48
tags: 
 - Java
 - RabbitMQ
categories:
 - MQ
 - RabbitMQ
---

# 04_simple简单队列

相关依赖：

```xml
<dependencies>
    <dependency>
        <groupId>com.rabbitmq</groupId>
        <artifactId>amqp-client</artifactId>
        <version>4.0.2</version>
    </dependency>

    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-api</artifactId>
        <version>1.7.10</version>
    </dependency>

    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-log4j12</artifactId>
        <version>1.7.5</version>
    </dependency>

    <dependency>
        <groupId>log4j</groupId>
        <artifactId>log4j</artifactId>
        <version>1.2.17</version>
    </dependency>

    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
    </dependency>

</dependencies>
```





---
## 模型
![简单队列模型](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/RabbitMQ/04%E7%AE%80%E5%8D%95%E9%98%9F%E5%88%97.png)
1. P:消息的生产者（Producer）
2. 中间：队列（queue）
3. C:消费者（Consumer）

---
## 获取MQ连接
定义ConnectionUtils工具类：
```java
/**
 * @author xieweidu
 * @createDate 2019-06-19 22:25
 */
public class ConnectionUtils {

    /**
     * 获取MQ的连接
     * @return
     */
    public static Connection getConnection() throws IOException, TimeoutException {
        // 定义一个连接工厂
        ConnectionFactory factory = new ConnectionFactory();
        // 设置服务地址
        factory.setHost("127.0.0.1");
        // AMQP协议
        // 端口
        factory.setPort(5672);
        // 设置vhost
        factory.setVirtualHost("/vhost_xwd");
        // 用户名
        factory.setUsername("user_xwd");
        // 密码
        factory.setPassword("user_xwd");
        return factory.newConnection();
    }
```

---
## 生产者生产消息
生产者Send类：
```java
/**
 * @author xieweidu
 * @createDate 2019-06-19 22:29
 */
public class Send {

    private static final String QUEUE_NAME = "test_simple_queue";

    public static void main(String[] args) throws IOException, TimeoutException {
        // 获取一个连接
        Connection connection = ConnectionUtils.getConnection();
        // 从连接中获取一个通道
        Channel channel = connection.createChannel();
        // 声明一个队列
        channel.queueDeclare(QUEUE_NAME,false,false,false,null);

        String msg = "hello simple!";

        channel.basicPublish("",QUEUE_NAME,null,msg.getBytes());

        System.out.println("-----send msg:"+msg);

        channel.close();

        connection.close();

    }

}

```

---
## 消费者消费消息
消费者Recv类（老版本）
```java
/**
 * 消息者获取消息
 * @author xieweidu
 * @createDate 2019-06-19 22:38
 */
public class OldConsumer {

    private static final String QUEUE_NAME = "test_simple_queue";

    public static void main(String[] args) throws IOException, TimeoutException, InterruptedException {
        // 获取连接
        Connection connection = ConnectionUtils.getConnection();
        // 创建频道
        Channel channel = connection.createChannel();

        // 老版本
        // 定义队列的消费者
        QueueingConsumer consumer = new QueueingConsumer(channel);
        // 监听队列
        channel.basicConsume(QUEUE_NAME,true,consumer);
        while (true){
            QueueingConsumer.Delivery delivery = consumer.nextDelivery();
            String msgString = new String(delivery.getBody());
            System.out.println("[OldComsumer]"+msgString);
        }



    }

}
```
（新版本）
```java
/**
 * @author xieweidu
 * @createDate 2019-06-19 22:48
 */
public class NewConsumer {

    private static final String QUEUE_NAME = "test_simple_queue";

    public static void main(String[] args) throws IOException, TimeoutException, InterruptedException {
        // 获取连接
        Connection connection = ConnectionUtils.getConnection();
        // 创建频道
        Channel channel = connection.createChannel();

        // 新版本
        // 队列声明
        channel.queueDeclare(QUEUE_NAME,false,false,false,null);
		// 定义一个消费者
        DefaultConsumer consumer = new DefaultConsumer(channel) {
            // 获取到 到达的消息就 干嘛干嘛
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                String msg = new String(body, "utf-8");
                System.out.println("新版本----消费：" + msg);

            }
        };
        // 监听队列  安卓相当于AddListener
        channel.basicConsume(QUEUE_NAME,true,consumer);

    }


}
```

