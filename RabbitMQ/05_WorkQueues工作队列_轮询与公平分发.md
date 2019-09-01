---
title: 05_WorkQueues工作队列_轮询与公平分发
date: 2019-06-19 ‏‎22:59:42
tags: 
 - Java
 - RabbitMQ
categories:
 - MQ
 - RabbitMQ
---

# 05_WorkQueues工作队列_轮询与公平分发



## 简单队列的不足

耦合性高，生产者——对应消费者

如果想要有多个消费者去消费队列里的信息，这时候就不行了

队列名变更，这时候得两边都要变更



---

## 模型：

![工作队列](https://raw.githubusercontent.com/tomxwd/ImageHosting/master/blog/RabbitMQ/05workQueues%E5%B7%A5%E4%BD%9C%E9%98%9F%E5%88%97.png)

一个生产者，多个消费者。

为什么会出现**工作队列**？

Simple队列是一一对应的，而且我们实际开发，生产者消费消息是毫不费力的，而消费者一般是跟业务相结合的，消费者接受到消息之后就需要处理，可能需要花费时间，这时候队列就会挤压很多消息。



---

## 工作队列----轮询分发：

生产者：

生产50个消息。

```java
/**
 * @author xieweidu
 * @createDate 2019-06-19 23:05
 */
public class Send {

    /**
     *                    \-----C1
     * p --- > QUEUE ---- \
     *                    \-----C2
     */

    private static final String QUEUE_NAME="test_work_queue";

    public static void main(String[] args) throws IOException, TimeoutException, InterruptedException {
        // 获取连接
        Connection conn = ConnectionUtils.getConnection();
        // 获取channel
        Channel channel = conn.createChannel();
        // 声明一个queue队列
        channel.queueDeclare(QUEUE_NAME,false,false,false,null);
        for (int i = 0; i < 50; i++) {
            String msg = "tom:"+i;
            System.out.println("WorkQueues send: "+msg);
            channel.basicPublish("",QUEUE_NAME,null,msg.getBytes());
            Thread.sleep(i*20);
        }
        channel.close();
        conn.close();

    }

}
```

消费者1：

```java
/**
 * @author xieweidu
 * @createDate 2019-06-19 23:11
 */
public class Consumer1 {

    private static final String QUEUE_NAME="test_work_queue";


    public static void main(String[] args) throws IOException, TimeoutException {

        // 获取连接
        final Connection connection = ConnectionUtils.getConnection();

        // 获取Channel
        final Channel channel = connection.createChannel();
        // 声明队列
        channel.queueDeclare(QUEUE_NAME,false,false,false,null);
        // 定义一个消费者
        final DefaultConsumer consumer = new DefaultConsumer(channel) {
            // 消息到达 触发方法
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                final String msg = new String(body, "utf-8");
                System.out.println("[1] Consumer1 msg: "+msg);
                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    System.out.println("[1] Consumer1 done");
                }
            }
        };
        channel.basicConsume(QUEUE_NAME,true,consumer);


    }

}

```

消费者2：

```java
/**
 * @author xieweidu
 * @createDate 2019-06-19 23:18
 */
public class Consumer2 {

    private static final String QUEUE_NAME="test_work_queue";


    public static void main(String[] args) throws IOException, TimeoutException {

        // 获取连接
        final Connection connection = ConnectionUtils.getConnection();

        // 获取Channel
        final Channel channel = connection.createChannel();
        // 声明队列
        channel.queueDeclare(QUEUE_NAME,false,false,false,null);
        // 定义一个消费者
        final DefaultConsumer consumer = new DefaultConsumer(channel) {
            // 消息到达 触发方法
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                final String msg = new String(body, "utf-8");
                System.out.println("[2] Consumer1 msg: "+msg);
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    System.out.println("[2] Consumer1 done");
                }
            }
        };
        channel.basicConsume(QUEUE_NAME,true,consumer);


    }
}
```

消费者1是2秒消费一个，消费者2是1秒消费一个。

结果：

消费者1：

```
[1] Consumer1 msg: tom:0
[1] Consumer1 done
[1] Consumer1 msg: tom:2
[1] Consumer1 done
[1] Consumer1 msg: tom:4
[1] Consumer1 done
[1] Consumer1 msg: tom:6
[1] Consumer1 done
[1] Consumer1 msg: tom:8
[1] Consumer1 done
[1] Consumer1 msg: tom:10
```

消费者2：

```
[2] Consumer2 msg: tom:1
[2] Consumer2 done
[2] Consumer2 msg: tom:3
[2] Consumer2 done
[2] Consumer2 msg: tom:5
[2] Consumer2 done
[2] Consumer2 msg: tom:7
[2] Consumer2 done
[2] Consumer2 msg: tom:9
[2] Consumer2 done
[2] Consumer2 msg: tom:11
....
```

现象上看，消费者1和消费者2处理的消息是一样条数的，虽然消费者2消费的比较快，但是他们两个消费者都是消费一样多的数据。

这种叫做**轮询分发（round-robin）**。

轮询分发的ack是true，自动应答。



---

## 工作队列----公平分发fair dipatch

使用basicQos(prefetch=1)

使用公平分发需要关闭自动应答ack，改为手动(false)。

#### 生产者

1. 每个消费者发送确认消息之前，消息队列不发送下一个消息到消费者，一次只处理一个消息。
   限制发送给同一个消费者不得超过一条消息。
2. prefetchCount=1，channel.basicQos(prefetchCount)，来表示一次一个。

代码：

```java
/**
 * @author xieweidu
 * @createDate 2019-06-19 23:05
 */
public class Send {

    /**
     *                    \-----C1
     * p --- > QUEUE ---- \
     *                    \-----C2
     */

    private static final String QUEUE_NAME="test_work_queue";

    public static void main(String[] args) throws IOException, TimeoutException, InterruptedException {
        // 获取连接
        Connection conn = ConnectionUtils.getConnection();
        // 获取channel
        Channel channel = conn.createChannel();
        // 声明一个queue队列
        channel.queueDeclare(QUEUE_NAME,false,false,false,null);

        /**
         * 每个消费者发送确认消息之前，消息队列不发送下一个消息到消费者，一次只处理一个消息
         *
         * 限制发送给同一个消费者不得超过一条消息
         */
        int prefetchCount = 1;
        channel.basicQos(prefetchCount);


        for (int i = 0; i < 50; i++) {
            String msg = "tom:"+i;
            System.out.println("WorkQueues send: "+msg);
            channel.basicPublish("",QUEUE_NAME,null,msg.getBytes());
            Thread.sleep(20);
        }
        channel.close();
        conn.close();

    }
```

#### 消费者1：

1. 首先也要设置

   ```java
           // 保证一次分发到一个
           int prefetchCount = 1;
           channel.basicQos(prefetchCount);
   ```

2. 其次手动回执：

   ```java
    // 手动回执
                       channel.basicAck(envelope.getDeliveryTag(),false);
   ```

3. 最后关闭自动应答

   ```java
   boolean autoAck = false; //自动应答
           channel.basicConsume(QUEUE_NAME,autoAck,consumer);
   ```

代码：

```java
/**
 * @author xieweidu
 * @createDate 2019-06-19 23:11
 */
public class Consumer1 {

    private static final String QUEUE_NAME="test_work_queue";


    public static void main(String[] args) throws IOException, TimeoutException {

        // 获取连接
        final Connection connection = ConnectionUtils.getConnection();

        // 获取Channel
        final Channel channel = connection.createChannel();
        // 声明队列
        channel.queueDeclare(QUEUE_NAME,false,false,false,null);

        // 保证一次分发到一个
        int prefetchCount = 1;
        channel.basicQos(prefetchCount);

        // 定义一个消费者
        final DefaultConsumer consumer = new DefaultConsumer(channel) {
            // 消息到达 触发方法
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                final String msg = new String(body, "utf-8");
                System.out.println("[1] Consumer1 msg: "+msg);
                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    System.out.println("[1] Consumer1 done");
                    // 手动回执
                    channel.basicAck(envelope.getDeliveryTag(),false);
                }
            }
        };
        boolean autoAck = false; //自动应答
        channel.basicConsume(QUEUE_NAME,autoAck,consumer);


    }

}
```



#### 消费者2：

消费者2同理消费者1，只是时间是一秒。

代码：

```java
/**
 * @author xieweidu
 * @createDate 2019-06-19 23:18
 */
public class Consumer2 {

    private static final String QUEUE_NAME="test_work_queue";


    public static void main(String[] args) throws IOException, TimeoutException {

        // 获取连接
        final Connection connection = ConnectionUtils.getConnection();

        // 获取Channel
        final Channel channel = connection.createChannel();
        // 声明队列
        channel.queueDeclare(QUEUE_NAME,false,false,false,null);

        // 保证一次分发到一个
        int prefetchCount = 1;
        channel.basicQos(prefetchCount);

        // 定义一个消费者
        final DefaultConsumer consumer = new DefaultConsumer(channel) {
            // 消息到达 触发方法
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                final String msg = new String(body, "utf-8");
                System.out.println("[2] Consumer2 msg: "+msg);
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    System.out.println("[2] Consumer2 done");
                    // 手动回执
                    channel.basicAck(envelope.getDeliveryTag(),false);
                }
            }
        };
        boolean autoAck = false; //自动应答
        channel.basicConsume(QUEUE_NAME,autoAck,consumer);


    }
}
```



## 结果：

生产者没变化。

消费者1：

```
[1] Consumer1 msg: tom:1
[1] Consumer1 done
[1] Consumer1 msg: tom:4
[1] Consumer1 done
[1] Consumer1 msg: tom:7
[1] Consumer1 done
[1] Consumer1 msg: tom:10
[1] Consumer1 done
[1] Consumer1 msg: tom:13
[1] Consumer1 done
[1] Consumer1 msg: tom:16
[1] Consumer1 done
...
```

消费者2：

```
[2] Consumer2 msg: tom:0
[2] Consumer2 done
[2] Consumer2 msg: tom:2
[2] Consumer2 done
[2] Consumer2 msg: tom:3
[2] Consumer2 done
[2] Consumer2 msg: tom:5
[2] Consumer2 done
[2] Consumer2 msg: tom:6
[2] Consumer2 done
[2] Consumer2 msg: tom:8
[2] Consumer2 done
[2] Consumer2 msg: tom:9
[2] Consumer2 done
[2] Consumer2 msg: tom:11
[2] Consumer2 done
[2] Consumer2 msg: tom:12
[2] Consumer2 done
[2] Consumer2 msg: tom:14
...
```

**消费者2处理的比消费者1多，简单说就是能者多劳，根据效率来分发消息**。

