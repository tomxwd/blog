---
title: 09_Routing模式
date: 2019-06-26 ‏‎‏‎22:31:08
tags: 
 - Java
 - RabbitMQ
categories:
 - MQ
 - RabbitMQ
---

# 09_Routing模式

> 路由模式

## 模型

![routing模型](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/RabbitMQ/09Routing%E6%A8%A1%E5%9E%8B.png)

## 生产者

```java
/**
 * @author xieweidu
 * @createDate 2019-06-26 22:33
 */
public class Send {

    public static final String EXCHANGE_NAME = "test_exchange_direct";

    public static void main(String[] args) throws IOException, TimeoutException {
        final Connection connection = ConnectionUtils.getConnection();
        final Channel channel = connection.createChannel();
        // Exchange
        channel.exchangeDeclare(EXCHANGE_NAME,"direct");

        final String msg = "hello direct";
        System.out.println("[send] : " + msg);

        String routingKey = "warning";
        channel.basicPublish(EXCHANGE_NAME,routingKey,null,msg.getBytes());

        channel.close();
        connection.close();

    }

}
```

改变routingKey的值可以改为“error”，“info”，“warning”。消费者1只会接收到error的信息，而消费者2可以接收到三种信息。



---

## 消费者1

```java
/**
 * @author xieweidu
 * @createDate 2019-06-26 22:39
 */
public class Consumer1 {

    public static final String EXCHANGE_NAME = "test_exchange_direct";
    public static final String QUEUE_NAME = "test_queue_direct_1";

    public static void main(String[] args) throws IOException, TimeoutException {
        final Connection connection = ConnectionUtils.getConnection();
        final Channel channel = connection.createChannel();

        channel.queueDeclare(QUEUE_NAME,false,false,false,null);

        channel.basicQos(1);

        channel.queueBind(QUEUE_NAME,EXCHANGE_NAME,"error");

        final DefaultConsumer consumer = new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                String msg = new String(body, "utf-8");
                System.out.println("[1 consumer] : " + msg);

                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    System.out.println("[1 consumer] : DONE");
                    channel.basicAck(envelope.getDeliveryTag(), false);
                }

            }
        };

        boolean autoAck = false;
        channel.basicConsume(QUEUE_NAME,autoAck,consumer);

    }

}
```

routingKey：“error”



---

## 消费者2

```java
/**
 * @author xieweidu
 * @createDate 2019-06-26 22:44
 */
public class Consumer2 {

    public static final String EXCHANGE_NAME = "test_exchange_direct";
    public static final String QUEUE_NAME = "test_queue_direct_2";

    public static void main(String[] args) throws IOException, TimeoutException {
        final Connection connection = ConnectionUtils.getConnection();
        final Channel channel = connection.createChannel();

        channel.queueDeclare(QUEUE_NAME,false,false,false,null);

        channel.queueBind(QUEUE_NAME,EXCHANGE_NAME,"error");
        channel.queueBind(QUEUE_NAME,EXCHANGE_NAME,"info");
        channel.queueBind(QUEUE_NAME,EXCHANGE_NAME,"warning");

        channel.basicQos(1);

        final DefaultConsumer consumer = new DefaultConsumer(channel){
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                String msg = new String(body,"utf-8");
                System.out.println("[2 consumer] : "+msg);
                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    System.out.println("[2 consumer] : DONE");
                    channel.basicAck(envelope.getDeliveryTag(),false);
                }
            }
        };

        channel.basicConsume(QUEUE_NAME,consumer);

    }

}
```

routingKey：“error”，“info”，“warning”；



## 结果

Consumer1只会收到error的信息，而Consumer2会收到error，info，warning的消息。

举例：为不同级别的人提供消息，有群发的，有低级不可见的等。

