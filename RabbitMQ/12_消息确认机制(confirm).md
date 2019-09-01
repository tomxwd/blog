---
title: 12_消息确认机制(confirm)
date: 2019-07-01 ‏‎‏‎‏‎17:44:07
tags: 
 - Java
 - RabbitMQ
categories:
 - MQ
 - RabbitMQ
---

# 12_消息确认机制(confirm)

> 生产者端confirm模式的实现原理

生产者将信道（channel）设置成confirm模式，所有在该信道发布的消息都会指派一个唯一的ID（标识这个消息的ID，从1开始），一旦消息被投递到匹配的队列之后，RabbitMQ的broker会发送一个确认信息（包含消息的唯一ID）给生产者，这就使得生产者知道已经正确到达目的队列，如果消息和队列是可持久化的，那么确认消息会将消息写到磁盘后发出。broker回传给生产者的确认消息中deliver-tag域包含了确认消息的序列号，此外broker也可以设置basic.ack的multiple域，表示这个序列号之前的所有消息都已经得到了处理。



Confirm模式最大的好处是在于他是异步的。一旦发布一条消息之后，生产者可以在等待确认消息返回的期间继续往RabbitMQ发送下面的消息。



当消息最终到达RabbitMQ之后，生产者可以通过会掉的方法来处理，如果RabbitMQ自身出现了异常，导致消息丢失，RabbitMQ就会发送一条nack消息。



#### 开启confirm模式：

channel.confirmSelect();



#### 编程模式：

1. 普通： 发送一条waitForConfirms()
2. 批量： 发送一批waitForConfirms()
3. 异步confirm模式： 提供一个回调方法，一条或者多条后会调用。  



---

## 编程模式——普通（单条）

#### 生产者

```java
/**
 * @author xieweidu
 * @createDate 2019-07-01 18:10
 * 普通模式
 */
public class Send1 {

    public static final String QUEUE_NAME = "test_queue_confirm1";

    public static void main(String[] args) throws IOException, TimeoutException, InterruptedException {
        final Connection connection = ConnectionUtils.getConnection();
        final Channel channel = connection.createChannel();
        channel.queueDeclare(QUEUE_NAME,false,false,false,null);

        // 生产者调用confirmSelect，将channel设置为confirm模式
        // 注意：同一个队列，channel不能设置为两种模式，一个用tx，一个用confirm会报错。
        channel.confirmSelect();

        final String msg = "HELLO CONFIRM1";

        channel.basicPublish("",QUEUE_NAME,null,msg.getBytes());

        if (!channel.waitForConfirms()){
            System.out.println("msg send failed");
        }else{
            System.out.println("msg send ok");
        }

        channel.close();
        connection.close();


    }
}
```

注意：同一个队列，channel不能设置为两种模式，一个用tx，一个用confirm会报错。



---

## 编程模式——批量（多条）

#### 生产者

```java
/**
 * @author xieweidu
 * @createDate 2019-07-01 18:10
 * 批量模式
 */
public class Send1 {

    public static final String QUEUE_NAME = "test_queue_confirm1";

    public static void main(String[] args) throws IOException, TimeoutException, InterruptedException {
        final Connection connection = ConnectionUtils.getConnection();
        final Channel channel = connection.createChannel();
        channel.queueDeclare(QUEUE_NAME,false,false,false,null);

        // 生产者调用confirmSelect，将channel设置为confirm模式
        // 注意：同一个队列，channel不能设置为两种模式，一个用tx，一个用confirm会报错。
        channel.confirmSelect();

        final String msg = "HELLO CONFIRM1";

        // 批量发送
        for (int i = 0; i < 10; i++) {
            channel.basicPublish("",QUEUE_NAME,null,msg.getBytes());
        }

        // 确认
        if (!channel.waitForConfirms()){
            System.out.println("msg send failed");
        }else{
            System.out.println("msg send ok");
        }

        channel.close();
        connection.close();


    }
}
```

批量模式即是批量发送完后再确认。



---

## 编程模式——异步模式

channel对象提供的ConfirmListener()回调方法只包括deliveryTag（当前channel发送的消息序号），我们需要自己为每一个Channel维护一个unconfirm的消息序号集合，每publish一条数据，集合中元素+1，每回调一次handleAck()方法，unconfirm集合删掉相应的一条(multiple=false)或多条(multiple=true)记录，从程序运行效率上看，这个unconfirm集合最好采用有序集合SortedSet存储结构。

```java
/**
 * @author xieweidu
 * @createDate 2019-07-01 19:04
 */
public class Send {

    public static final String QUEUE_NAME = "test_queue_confirm_async";

    public static void main(String[] args) throws IOException, TimeoutException {
        final Connection connection = ConnectionUtils.getConnection();
        final Channel channel = connection.createChannel();

        channel.queueDeclare(QUEUE_NAME,false,false,false,null);

        // confirm模式开启
        channel.confirmSelect();

        // 存放未确认的消息的标识序号
        final SortedSet<Long> confirmSet = Collections.synchronizedSortedSet(new TreeSet<Long>());

        // 通道添加监听
        channel.addConfirmListener(new ConfirmListener() {
            // 没有问题的handleAck
            public void handleAck(long deliveryTag, boolean multiple) throws IOException {
                if(multiple){
                    System.out.println("----Ack----multiple");
                    confirmSet.headSet(deliveryTag+1).clear();
                }else{
                    System.out.println("----Ack----multiple = false");
                    confirmSet.remove(deliveryTag);
                }
            }
            // 有问题的handleNack （如重发 重试等）
            public void handleNack(long deliveryTag, boolean multiple) throws IOException {
                if(multiple){
                    System.out.println("----Nack----multiple");
                    confirmSet.headSet(deliveryTag+1).clear();
                }else{
                    System.out.println("----Nack----multiple");
                    confirmSet.remove(deliveryTag);
                }
            }
        });

        String msg = "async-msg";

        for (int i = 0; i < 100; i++) {
            final long seqNo = channel.getNextPublishSeqNo();
            System.out.println("seqNo = " + seqNo);
            channel.basicPublish("",QUEUE_NAME,null,msg.getBytes());
            confirmSet.add(seqNo);
        }


    }

}
```

控制台输出：

```
seqNo = 1
seqNo = 2
seqNo = 3
seqNo = 4
seqNo = 5
seqNo = 6
seqNo = 7
seqNo = 8
seqNo = 9
seqNo = 10
seqNo = 11
seqNo = 12
seqNo = 13
seqNo = 14
seqNo = 15
seqNo = 16
seqNo = 17
seqNo = 18
seqNo = 19
seqNo = 20
seqNo = 21
seqNo = 22
seqNo = 23
seqNo = 24
seqNo = 25
seqNo = 26
seqNo = 27
seqNo = 28
seqNo = 29
seqNo = 30
seqNo = 31
seqNo = 32
seqNo = 33
seqNo = 34
seqNo = 35
seqNo = 36
seqNo = 37
seqNo = 38
----Ack----multiple
seqNo = 39
seqNo = 40
seqNo = 41
----Ack----multiple
seqNo = 42
seqNo = 43
seqNo = 44
seqNo = 45
seqNo = 46
seqNo = 47
seqNo = 48
seqNo = 49
seqNo = 50
seqNo = 51
seqNo = 52
seqNo = 53
seqNo = 54
seqNo = 55
seqNo = 56
seqNo = 57
seqNo = 58
seqNo = 59
seqNo = 60
seqNo = 61
seqNo = 62
seqNo = 63
seqNo = 64
seqNo = 65
seqNo = 66
seqNo = 67
seqNo = 68
seqNo = 69
seqNo = 70
seqNo = 71
seqNo = 72
seqNo = 73
seqNo = 74
seqNo = 75
seqNo = 76
seqNo = 77
seqNo = 78
seqNo = 79
seqNo = 80
seqNo = 81
seqNo = 82
seqNo = 83
seqNo = 84
seqNo = 85
seqNo = 86
seqNo = 87
seqNo = 88
seqNo = 89
seqNo = 90
seqNo = 91
seqNo = 92
seqNo = 93
----Ack----multiple
seqNo = 94
seqNo = 95
seqNo = 96
----Ack----multiple
----Ack----multiple
seqNo = 97
seqNo = 98
seqNo = 99
----Ack----multiple
----Ack----multiple
----Ack----multiple = false
----Ack----multiple = false
----Ack----multiple = false
----Ack----multiple
----Ack----multiple
----Ack----multiple
----Ack----multiple
----Ack----multiple
----Ack----multiple
seqNo = 100
----Ack----multiple
----Ack----multiple
----Ack----multiple = false
----Ack----multiple = false
```

这里可以通过结果看到：有时候是回一条，有时候是多条。不管他回来是多少条，多条就按多条回滚的代码，单条就按单条来处理，比如3s后再发送等。具体的看业务。

视频讲解不够仔细，需要补充。