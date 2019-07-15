---
title: 10_Topic主题模式
data: 2019-07-01 ‏‎‏‎‏‎16:59:08
tags: 
 - Java
 - RabbitMQ
categories:
 - MQ
 - RabbitMQ
---

# 10_Topic主题模式

## 模型

![Topic主题模式模型](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/RabbitMQ/10Topic%E6%A8%A1%E5%9E%8B.png)



---

## 完成一个小功能

商品：发布 删除 修改 查询......



---

#### 生产者

```java
/**
 * @author xieweidu
 * @createDate 2019-07-01 17:05
 */
public class Send {

    public static final String EXCHANGE_NAME = "test_exchange_topic";

    public static void main(String[] args) throws IOException, TimeoutException {
        final Connection connection = ConnectionUtils.getConnection();
        final Channel channel = connection.createChannel();
        channel.exchangeDeclare(EXCHANGE_NAME,"topic");
        String msg1 = "商品添加。。。。";
        String msg2 = "商品删除。。。。";
        String msg3 = "商品修改。。。。";
        String msg = msg3;

        String key1 = "goods.add";
        String key2 = "goods.delete";
        String key3 = "goods.edit";

        String routingKey = key3;

        channel.basicPublish(EXCHANGE_NAME,routingKey,null,msg.getBytes());

        System.out.println("[send] : " + msg);

        channel.close();
        connection.close();

    }


}
```

可以根据msg和routingKey的值发送不同的信息。



---

## 消费者1

```java
/**
 * @author xieweidu
 * @createDate 2019-07-01 17:08
 */
public class Consumer1 {

    public static final String EXCHANGE_NAME = "test_exchange_topic";
    public static final String QUEUE_NAME = "test_queue_topic_1";

    public static void main(String[] args) throws IOException, TimeoutException {

        final Connection connection = ConnectionUtils.getConnection();

        final Channel channel = connection.createChannel();

        channel.queueDeclare(QUEUE_NAME,false,false,false,null);

        channel.queueBind(QUEUE_NAME,EXCHANGE_NAME,"goods.add");

        channel.basicQos(1);

        final DefaultConsumer consumer = new DefaultConsumer(channel){
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                final String msg = new String(body, "utf-8");
                System.out.println("[1 Consumer] : "+msg);
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    System.out.println("[1 Consumer] DONE!");
                    channel.basicAck(envelope.getDeliveryTag(),false);
                }
            }
        };

        channel.basicConsume(QUEUE_NAME,consumer);


    }

}
```

消费者1只能接受routingKey为“goods.add"的消息。



---

## 消费者2

```java
/**
 * @author xieweidu
 * @createDate 2019-07-01 17:08
 */
public class Consumer2 {

    public static final String EXCHANGE_NAME = "test_exchange_topic";
    public static final String QUEUE_NAME = "test_queue_topic_2";

    public static void main(String[] args) throws IOException, TimeoutException {

        final Connection connection = ConnectionUtils.getConnection();

        final Channel channel = connection.createChannel();

        channel.queueDeclare(QUEUE_NAME,false,false,false,null);

        channel.queueBind(QUEUE_NAME,EXCHANGE_NAME,"goods.#");

        channel.basicQos(1);

        final DefaultConsumer consumer = new DefaultConsumer(channel){
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                final String msg = new String(body, "utf-8");
                System.out.println("[2 Consumer] : "+msg);
                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    System.out.println("[2 Consumer] DONE!");
                    channel.basicAck(envelope.getDeliveryTag(),false);
                }
            }
        };

        channel.basicConsume(QUEUE_NAME,consumer);


    }

}
```

消费者2可以接受已”goods."开头的所有消息。