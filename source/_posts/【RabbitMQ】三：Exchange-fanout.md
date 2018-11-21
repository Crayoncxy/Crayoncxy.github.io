---
layout: posts
title: 【RabbitMQ】三：Exchange-fanout
date: 2018-11-21 14:23:03
tags:
- RabbitMQ
categories: 学习笔记
---

# 一、概述

&ensp;&ensp;&ensp;&ensp;上一篇文章中讲述了一个简单的消息传递模型，消息从生产者发送到消费者再发送到队列，实际的工作中生产者不知道要把消息发送给哪个队列，可能有多个消费者要生产者的消息，也可能有的消费者不需要生产者的全部消息，比如日志系统，一个消费者需要info级别的信息，另一个消费者需要error和info级别的信息，这时候我们就用到了交换机，生产者把消息发送到交换机，交换机像是一个消息中转站，一边接收生产者的消息，一边将消息根据绑定队列的路由规则发送给指定的队列，这种我们称作“发布/订阅”模式。

![png1]([RabbitMQ]三：Exchange-fanout/png1.png)

# 二、交换机

&ensp;&ensp;&ensp;&ensp;这是这篇文章的主题，在学习笔记一种有说到，一共有四种交换机，fanout（分发），topic（匹配），direct（直连），header（主题），这篇文章中，我们先探讨fanout类型的交换机。

我们可以先从管理台上看一下RabbitMQ为我们提供的交换机，如下图：

![png2]([RabbitMQ]三：Exchange-fanout/png2.png)

图上所有amq.*的交换机，都是RabbitMQ为我们创建的交换机。

&ensp;&ensp;&ensp;&ensp;**匿名交换机：**这里有个概念要说明一下，在上篇文章中我们发送消息时并没有使用交换机，但是消息依然发送到了指定的队列，这是因为RabbitMQ为我们创建了一个“”空字符串的匿名交换机，我们看下上篇文章中的发送消息的代码

```java
channel.basicPublish("",QUEUE_NAME,null,message.getBytes("UTF-8"));
```

我们在Channel类中查看一下这个函数的声明参数类型及含义 

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

&ensp;&ensp;&ensp;&ensp;可以看到第一个参数就是交换机，第二个参数是路由规则，当时写的时候我还好奇，为什么路由规则这个参数我们传参传的是队列的名称。原来是因为RabbitMQ为我们创建了一个“”空字符串的匿名交换机，此时，消息默认发送到路由规则同名的队列上，这也就是上篇文章中消息能够成功发送到我们指定队列上的原因。

# 三、临时队列

&ensp;&ensp;&ensp;&ensp;上一篇文章中我们定义了一个静态的成员变量QUEUE_NAME，指定了队列的名字为HelloMq，RabbitMQ为我们创建了这个队列。实际工作中，我们可能不需要指定队列的名字，或者说我们不需要使用已经存在的队列，这时候我们就可以使用RabbitMQ创建的临时队列。每当我们连接到RabbitMQ时，都创建一个新的空队列，并且让RabbitMQ随机选择一个名字给我们，并且在所有消费者断开的时候，队列自动删除。

```java
String queueName = channel.queueDeclare().getQueue();
```

JAVA中使用queueDeclare().getQueue()方法创建一个随机的非持久化队列。

# 四、路由规则

&ensp;&ensp;&ensp;&ensp;前边我们说到交换机根据路由规则发送消息到指定队列，这里的功能实现是队列与交换机之间事先绑定了一个路由规则，当消息发送过来的时候，交换机根据路由规则匹配到相应的队列上。队列与交换机之间绑定路由规则代码如下：

```java
channel.queueBind(queueName, EXCHANGE_NAME, "routingKey")；
```

&ensp;&ensp;&ensp;&ensp;第一个参数是队列的名称，第二个参数是交换机的名称，第三个参数是要绑定的路由规则，可以根据不同的队列定义不同的绑定规则而实现不同的消息发送到不同的队列上。当然，本文中学习到的fanout类型的交换机会自动忽视绑定规则而将消息发送到所有与交换机绑定的队列上去。

## 五、源代码

本示例采用文章开头的配图结构，一个生产者，一个交换机，两个随机临时队列，两个消费者。

&ensp;&ensp;&ensp;&ensp;先写消息生产者，由消息生产者创建一个名为“HelloMq”的fanout类型交换机，我们可以看到发送消息的方法第一个参数变为交换机的名称了，由于交换机类型为fanout，会忽略绑定规则，因此第二个参数为空。

```java
package com.cn.chenxyt.mq;
 
import java.io.IOException;
 
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.Channel;
 
 
public class MqProducer {
	 public final static String EXCHANGE_NAME="HelloMq";
	 public static void main(String[] args) throws IOException, InterruptedException {
		 //创建连接工厂
		 ConnectionFactory factory = new ConnectionFactory();
		 //设置主机、用户名、密码和客户端端口号
		 factory.setHost("localhost");
		 factory.setUsername("guest");
		 factory.setPassword("guest");
		 factory.setPort(5672);
		 //创建一个新的连接 即TCP连接
		 Connection connection = factory.newConnection();
		 //创建一个通道
		 Channel channel = connection.createChannel();
		 //创建一个交换机
		 channel.exchangeDeclare(EXCHANGE_NAME,"fanout");
		 String message = "Hello World";
		 while (true){
			 //发送消息
			 channel.basicPublish(EXCHANGE_NAME,"",null,message.getBytes("UTF-8"));
	         System.out.println("Producer Send +'" + message + "!");
	         Thread.sleep(2000);
		 }
	}
}
```

&ensp;&ensp;&ensp;&ensp;接下来我们创建消费者1，消费者1声明了一个随机临时队列，并绑定到了交换机上，如前边所述，交换机类型为fanout，因此绑定规则为空。

```java
package com.cn.chenxyt.mq;
 
import java.io.IOException;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.Consumer;
import com.rabbitmq.client.DefaultConsumer;
import com.rabbitmq.client.Envelope;
import com.rabbitmq.client.AMQP.BasicProperties;
public class MqConsumer1 {
	 public final static String EXCHANGE_NAME="HelloMq";
	 public static void main(String[] args) throws IOException {
		 //创建连接工厂
		 ConnectionFactory factory = new ConnectionFactory();
		 //设置主机
		 factory.setHost("localhost");
		 //创建一个新的连接 即TCP连接
		 Connection connection = factory.newConnection();
		 //创建一个通道
		 Channel channel = connection.createChannel();
		 //创建随机队列
		 String queueName = channel.queueDeclare().getQueue();
		 //绑定队列到交换机
		 channel.queueBind(queueName, EXCHANGE_NAME, "");
	     System.out.println("Consumer1 Waiting Received messages");
	     //DefaultConsumer类实现了Consumer接口，通过传入一个channel，
	     //告诉服务器我们需要哪个channel的消息并监听channel，如果channel中有消息，就会执行回调函数handleDelivery
	     Consumer consumer = new DefaultConsumer(channel) {
	            @Override
	            public void handleDelivery(String consumerTag, Envelope envelope,
	                                       BasicProperties properties, byte[] body)
	                    throws IOException {
	                String message = new String(body, "UTF-8");
	                System.out.println("Consumer1 Received '" + message + "'");
	            }
	        };
	        //自动回复队列应答 -- RabbitMQ中的消息确认机制
	        channel.basicConsume(queueName, true, consumer);
	}
}
```

&ensp;&ensp;&ensp;&ensp;接下来创建消费者2，消费者2与消费者1相同，各自创建了一个随机临时队列并绑定在交换机上，为了便于区分，打印日志处区分了名字。

```java
package com.cn.chenxyt.mq;
 
import java.io.IOException;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.Consumer;
import com.rabbitmq.client.DefaultConsumer;
import com.rabbitmq.client.Envelope;
import com.rabbitmq.client.AMQP.BasicProperties;
public class MqConsumer2 {
	 public final static String EXCHANGE_NAME="HelloMq";
	 public static void main(String[] args) throws IOException {
		 //创建连接工厂
		 ConnectionFactory factory = new ConnectionFactory();
		 //设置主机
		 factory.setHost("localhost");
		 //创建一个新的连接 即TCP连接
		 Connection connection = factory.newConnection();
		 //创建一个通道
		 Channel channel = connection.createChannel();
		 //创建随机队列
		 String queueName = channel.queueDeclare().getQueue();
		 //绑定队列到交换机
		 channel.queueBind(queueName, EXCHANGE_NAME, "");
	     System.out.println("Consumer2 Waiting Received messages");
	     //DefaultConsumer类实现了Consumer接口，通过传入一个channel，
	     //告诉服务器我们需要哪个channel的消息并监听channel，如果channel中有消息，就会执行回调函数handleDelivery
	     Consumer consumer = new DefaultConsumer(channel) {
	            @Override
	            public void handleDelivery(String consumerTag, Envelope envelope,
	                                       BasicProperties properties, byte[] body)
	                    throws IOException {
	                String message = new String(body, "UTF-8");
	                System.out.println("Consumer2 Received '" + message + "'");
	            }
	        };
	        //自动回复队列应答 -- RabbitMQ中的消息确认机制
	        channel.basicConsume(queueName, true, consumer);
	}
}
```

&ensp;&ensp;&ensp;&ensp;接下来我们分别启动消费者1和消费者2，使他们处在监听状态。

![png3]([RabbitMQ]三：Exchange-fanout/png3.png)

![png4]([RabbitMQ]三：Exchange-fanout/png4.png)

&ensp;&ensp;&ensp;&ensp;同时我们在RabbitMQ的管理台可以看到新建了两条名字随机由RabbitMQ生成的队列，并且当消费者1和消费者2服务断开时，这两个队列会被删除

![png5]([RabbitMQ]三：Exchange-fanout/png5.png)

&ensp;&ensp;&ensp;&ensp;接下来我们启动消息生产者，每2s发送一条消息

![png6]([RabbitMQ]三：Exchange-fanout/png6.png)

&ensp;&ensp;&ensp;&ensp;同时可以看到消费者1和消费者2都收到了相同的消息，并且RabbitMQ管理台新建了一个名为“HelloMq”类型为"fanout"的交换机 

![png7]([RabbitMQ]三：Exchange-fanout/png7.png)

![png8]([RabbitMQ]三：Exchange-fanout/png8.png)

![png9]([RabbitMQ]三：Exchange-fanout/png9.png)

&ensp;&ensp;&ensp;&ensp;如上所示，我们就完成了使用fanout类型的交换机进行消息的“发布/订阅”，总结一下，我们可以使用fanout类型的交换机进行消息的广播发送。

# 六、代码下载

[下载地址](<https://pan.baidu.com/s/1dGcdsCL>)