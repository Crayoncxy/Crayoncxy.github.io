---
layout: posts
title: 【RabbitMQ】七：交换机、队列、消息的持久化
date: 2018-11-22 09:34:12
tags:
- RabbitMQ
categories: 学习笔记
---

# 一、概述

&ensp;&ensp;&ensp;&ensp;在生产过程中，难免会发生服务器宕机的事情，RabbitMQ也不例外，可能由于某种特殊情况下的异常而导致RabbitMQ宕机从而重启，那么这个时候对于消息队列里的数据，包括交换机、队列以及队列中存在消息恢复就显得尤为重要了。RabbitMQ本身带有持久化机制，包括交换机、队列以及消息的持久化。持久化的主要机制就是将信息写入磁盘，当RabbtiMQ服务宕机重启后，从磁盘中读取存入的持久化信息，恢复数据。(当然凡是都不是100%的，只能尽最大程度的保证消息不会丢失吧)

# 二、交换机的持久化

&ensp;&ensp;&ensp;&ensp;在前面的示例中，我们使用常规的声明交换机的方法

```java
		 channel.exchangeDeclare(EXCHANGE_NAME,"fanout");
```

&ensp;&ensp;&ensp;&ensp;使用这种方法声明的交换机，默认不是持久化的，在服务器重启之后，交换机会消失。我们在管理台的Exchange页签下查看交换机，可以看到使用上述方法声明的交换机，Features一列是空的，即没有任何附加属性。

![png1]([RabbitMQ]七：交换机、队列、消息的持久化/png1.png)

我们换用另一种方法声明交换机

```java
		 channel.exchangeDeclare(EXCHANGE_NAME, "fanout",true);
```

查看一下方法的说明

```java
    /**
     * Actively declare a non-autodelete exchange with no extra arguments
     * @see com.rabbitmq.client.AMQP.Exchange.Declare
     * @see com.rabbitmq.client.AMQP.Exchange.DeclareOk
     * @param exchange the name of the exchange
     * @param type the exchange type
     * @param durable true if we are declaring a durable exchange (the exchange will survive a server restart)
     * @throws java.io.IOException if an error is encountered
     * @return a declaration-confirm method to indicate the exchange was successfully declared
     */
    Exchange.DeclareOk exchangeDeclare(String exchange, String type, boolean durable) throws IOException;
```

&ensp;&ensp;&ensp;&ensp;我们可以看到第三个参数durable，如果为true时则表示要做持久化，当服务重启时，交换机依然存在，所以使用该方法声明的交换机是下面这个样子的（做测试的时候，需要先在管理台删掉原来的同名交换机）D表示durable，鼠标放在上边会显示为true

![png2]([RabbitMQ]七：交换机、队列、消息的持久化/png2.png)

&ensp;&ensp;&ensp;&ensp;现在重启RabbitMQ服务之后，可以看到我们声明的交换机仍然存在。

# 三、队列的持久化

&ensp;&ensp;&ensp;&ensp;与交换机的持久化相同，队列的持久化也是通过durable参数实现的，默认生成的随机队列不是持久化的。前面示例中声明的带有我们自定义名字的队列都是持久化的。

```java
		 channel.queueDeclare(QUEUE_NAME, true, false, false, null);
```

看一下方法的定义 

```java
    /**
     * Declare a queue
     * @see com.rabbitmq.client.AMQP.Queue.Declare
     * @see com.rabbitmq.client.AMQP.Queue.DeclareOk
     * @param queue the name of the queue
     * @param durable true if we are declaring a durable queue (the queue will survive a server restart)
     * @param exclusive true if we are declaring an exclusive queue (restricted to this connection)
     * @param autoDelete true if we are declaring an autodelete queue (server will delete it when no longer in use)
     * @param arguments other properties (construction arguments) for the queue
     * @return a declaration-confirm method to indicate the queue was successfully declared
     * @throws java.io.IOException if an error is encountered
     */
    Queue.DeclareOk queueDeclare(String queue, boolean durable, boolean exclusive, boolean autoDelete,
                                 Map<String, Object> arguments) throws IOException;
```

&ensp;&ensp;&ensp;&ensp;第二个参数跟交换机方法的参数一样，true表示做持久化，当RabbitMQ服务重启时，队列依然存在。这里说一下后边的三个参数，exclusive是排他队列，如果一个队列被声明为排他队列，那么这个队列只能被第一次声明他的连接所见，并在连接断开的时候自动删除。这里有三点需要说明，1、同一个连接的不同channel，是可以访问同一连接下创建的排他队列的。2、排他队列只能被声明一次，其他连接不允许声明同名的排他队列。3、及时排他队列是持久化的，当连接断开或者客户端退出时，排他队列依然会被删除。autoDelete是自动删除，为true时，当没有任何消费者订阅该队列时，队列会被自动删除。arguments：其它参数

# 四、消息持久化

&ensp;&ensp;&ensp;&ensp;消息的持久化是指当消息从交换机发送到队列之后，被消费者消费之前，服务器突然宕机重启，消息仍然存在。消息持久化的前提是队列持久化，假如队列不是持久化，那么消息的持久化毫无意义。

 通过如下代码设置消息的持久化

```java
channel.basicPublish(EXCHANGE_NAME,"",MessageProperties.PERSISTENT_TEXT_PLAIN,message.getBytes());
```

其中MessageProperties.PERSISTENT_TEXT_PLAIN是设置持久化的参数

我们查看basicPublish方法的定义

```java
    /**
     * Publish a message
     * @see com.rabbitmq.client.AMQP.Basic.Publish
     * @param exchange the exchange to publish the message to
     * @param routingKey the routing key
     * @param props other properties for the message - routing headers etc
     * @param body the message body
     * @throws java.io.IOException if an error is encountered
     */
    void basicPublish(String exchange, String routingKey, BasicProperties props, byte[] body) throws IOException;
```

在看下BasicProperties的类型

```java
        public BasicProperties(
            String contentType,
            String contentEncoding,
            Map<String,Object> headers,
            Integer deliveryMode,
            Integer priority,
            String correlationId,
            String replyTo,
            String expiration,
            String messageId,
            Date timestamp,
            String type,
            String userId,
            String appId,
            String clusterId)
```

&ensp;&ensp;&ensp;&ensp;其中deliveryMode是设置消息持久化的参数，等于1不设置持久化，等于2设置持久化。PERSISTENT_TEXT_PLAIN是实例化的一个deliveryMode=2的对象，便于编程。

```java
public static final BasicProperties PERSISTENT_TEXT_PLAIN =
    new BasicProperties("text/plain",
                        null,
                        null,
                        2,
                        0, null, null, null,
                        null, null, null, null,
                        null, null);
```

设置了队列的持久化和消息的持久化之后，当服务器宕机重启，存在队列中未发送的消息会依然存在。

&ensp;&ensp;&ensp;&ensp;以上就是关于RabbitMQ中持久化的一些内容，但是并不会严格的100%保证信息不会丢失，相关内容后续再说明。

# 五、总结

&ensp;&ensp;&ensp;&ensp;RabbitMQ的持久化有交换机、队列、消息的持久化。用于防止服务器宕机重启之后数据的丢失，其中交换机和队列的持久化都是设置durable参数为true，消息的持久化是设置Properties为MessageProperties.PERSITANT_TEXT_PLAIN,消息的持久化基于队列的持久化。持久化不是100%完全保证消息的可靠性。