---
title: 07_Publish-Subscribe订阅模式
date: 2019-06-19 23:33:05
tags: 
 - Java
 - RabbitMQ
categories:
 - MQ
 - RabbitMQ
---

# 07_Publish-Subscribe订阅模式

## 模型：

![订阅模式模型](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/RabbitMQ/07%E8%AE%A2%E9%98%85%E6%A8%A1%E5%BC%8F%E6%A8%A1%E5%9E%8B.png)

解读：

1. 一个生产者，多个消费者

2. 每一个消费者都有自己的队列

3. 生产者没有直接把消息发送到队列，而是发送到交换机（转发器exchange）

4. 每个队列都要绑定到交换机上

5. 生产者发送的消息经过交换机到达队列，就能实现一个消息被多个消费者消费。

   

   	|---->邮件

注册---|

			|---->短信



---

## 生产者

```java
/**
 * @author xieweidu
 * @createDate 2019-06-25 23:05
 */
public class Send {

    public static final String EXCHANGE_NAME = "test_exchange_fanout";

    public static void main(String[] args) throws IOException, TimeoutException {
        final Connection connection = ConnectionUtils.getConnection();

        final Channel channel = connection.createChannel();

        // 声明交换机
        channel.exchangeDeclare(EXCHANGE_NAME,"fanout");//分发

        // 发送消息
        final String msg = "hello ps";

        channel.basicPublish(EXCHANGE_NAME,"",null,msg.getBytes());

        System.out.println("Send: "+msg);

        channel.close();
        connection.close();

    }
}
```

![交换机](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/RabbitMQ/07%E4%BA%A4%E6%8D%A2%E6%9C%BA.png)

只有看到exchange，没有看到消息。

消息哪去了？丢失了！

因为交换机没有存储的能力，在rabbitmq中只有队列有存储能力。

因为这时候还没有队列绑定到这个交换机，所以数据丢失了。



---

## 消费者1

```java
/**
 * @author xieweidu
 * @createDate 2019-06-25 23:16
 */
public class Consumer1 {

    public static final String QUEUE_NAME = "test_queue_fanout_email";
    public static final String EXCHANGE_NAME = "test_exchange_fanout";


    public static void main(String[] args) throws IOException, TimeoutException {
        final Connection connection = ConnectionUtils.getConnection();
        final Channel channel = connection.createChannel();

        // 队列声明
        channel.queueDeclare(QUEUE_NAME,false,false,false,null);

        // 绑定队列到交换机
        channel.queueBind(QUEUE_NAME,EXCHANGE_NAME,"");

        channel.basicQos(1);

        // 定义一个消费者
        final DefaultConsumer consumer = new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                System.out.println("[1 consumer]:" + new String(body,"utf-8"));
                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    System.out.println("[1 consumer]: DONE");
                    channel.basicAck(envelope.getDeliveryTag(),false);
                }
            }
        };

        boolean autoAck = false;
        channel.basicConsume(QUEUE_NAME,autoAck,consumer);


    }
}
```





---

## 消费者2

```java
/**
 * @author xieweidu
 * @createDate 2019-06-25 23:16
 */
public class Consumer2 {

    public static final String QUEUE_NAME = "test_queue_fanout_sms";
    public static final String EXCHANGE_NAME = "test_exchange_fanout";


    public static void main(String[] args) throws IOException, TimeoutException {
        final Connection connection = ConnectionUtils.getConnection();
        final Channel channel = connection.createChannel();

        // 队列声明
        channel.queueDeclare(QUEUE_NAME,false,false,false,null);

        // 绑定队列到交换机
        channel.queueBind(QUEUE_NAME,EXCHANGE_NAME,"");

        channel.basicQos(1);

        // 定义一个消费者
        final DefaultConsumer consumer = new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                System.out.println("[2 consumer]:" + new String(body,"utf-8"));
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    System.out.println("[2 consumer]: DONE");
                    channel.basicAck(envelope.getDeliveryTag(),false);
                }
            }
        };

        boolean autoAck = false;
        channel.basicConsume(QUEUE_NAME,autoAck,consumer);


    }
}
```

![bindings绑定](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/RabbitMQ/07bindings%E7%BB%91%E5%AE%9A.png)

运行两个消费者后，发送一条消息，两个消费者结果如下：

```
[1 consumer]:hello ps
[1 consumer]: DONE
```

```
[2 consumer]:hello ps
[2 consumer]: DONE
```
