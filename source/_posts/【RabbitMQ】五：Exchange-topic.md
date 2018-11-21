---
layout: posts
title: 【RabbitMQ】五：Exchange-topic
date: 2018-11-21 15:48:42
tags:
- RabbitMQ
categories: 学习笔记
---

# 一、概述

&ensp;&ensp;&ensp;&ensp;前两篇文章中讲述了全部匹配和直接匹配两种交换机Exchange的类型，这篇文章讲述第三种模糊匹配Topic类型的交换机。

&ensp;&ensp;&ensp;&ensp;Topic类型的交换机，支持路由规则routingKey模糊匹配，匹配符号有‘#’全部一个或多个单词，‘*’一个单词。

&ensp;&ensp;&ensp;&ensp;示例：假设我们有两条队列，队列的绑定规则不确定，只有部分是确定的，也就是我们会接收包含这确定路由规则的信息。

&ensp;&ensp;&ensp;&ensp;实现方式：使用Topic类型的交换机，定义路由规则带有匹配符号‘#’或者‘*’,如图，为了更好的说明‘*’和‘#’之间的关系，我们定义如下四个队列分别绑定四种绑定规则com.chen.#  #.com.chen  com.chen.*和*.com.chen 。预期功能，当传入消息绑定com.chen.me时，能被13消费者收到，当传入消息绑定me.com.chen时，能被24消费者收到，当传入消息绑定com.chen.its.me时，能被消费者1收到，当传入消息绑定its.me.com.chen时，能被2消费者收到。

![png1]([RabbitMQ]五：Exchange-topic/png1.png)

# 二、源代码

&ensp;&ensp;&ensp;&ensp;与前面两篇相同，先写生产者，创建Topic类型的交换机，发送四条消息，带有上述四个routingKey

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
		 channel.exchangeDeclare(EXCHANGE_NAME,"topic");
		 String message1 = "我是com.chen.its.me";
		 String message2 = "我是its.me.com.chen";
		 String message3 = "我是com.chen.me";
		 String message4 = "我是me.com.chen";
		 channel.basicPublish(EXCHANGE_NAME,"com.chen.its.me",null,message1.getBytes("UTF-8"));
		 channel.basicPublish(EXCHANGE_NAME,"its.me.com.chen",null,message2.getBytes("UTF-8"));
		 channel.basicPublish(EXCHANGE_NAME,"com.chen.me",null,message3.getBytes("UTF-8"));
		 channel.basicPublish(EXCHANGE_NAME,"me.com.chen",null,message4.getBytes("UTF-8"));
	}
}
```

&ensp;&ensp;&ensp;&ensp;编写消费者1代码，由于我们要先启动消费者，所以消费者里也创建一个交换机，（生产者即可不用创建），新建一个随机队列，绑定com.chen.#路由规则。

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
		 //创建一个交换机
		 channel.exchangeDeclare(EXCHANGE_NAME,"topic");
		 //绑定队列到交换机
		 channel.queueBind(queueName, EXCHANGE_NAME, "com.chen.#");
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

新建消费者2，与消费者1类似，交换机只要创建一次即可，这里不保证哪个先启动，所以都声明了交换机。新建随机队列，绑定路由规则#.com.chen

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
		 //创建一个交换机
		 channel.exchangeDeclare(EXCHANGE_NAME,"topic");
		 //绑定队列到交换机
		 channel.queueBind(queueName, EXCHANGE_NAME, "#.com.chen");
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

新建消费者3，绑定路由规则com.chen.*

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
public class MqConsumer3 {
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
		 //创建一个交换机
		 channel.exchangeDeclare(EXCHANGE_NAME,"topic");
		 //绑定队列到交换机
		 channel.queueBind(queueName, EXCHANGE_NAME, "com.chen.*");
	     System.out.println("Consumer3 Waiting Received messages");
	     //DefaultConsumer类实现了Consumer接口，通过传入一个channel，
	     //告诉服务器我们需要哪个channel的消息并监听channel，如果channel中有消息，就会执行回调函数handleDelivery
	     Consumer consumer = new DefaultConsumer(channel) {
	            @Override
	            public void handleDelivery(String consumerTag, Envelope envelope,
	                                       BasicProperties properties, byte[] body)
	                    throws IOException {
	                String message = new String(body, "UTF-8");
	                System.out.println("Consumer3 Received '" + message + "'");
	            }
	        };
	        //自动回复队列应答 -- RabbitMQ中的消息确认机制
	        channel.basicConsume(queueName, true, consumer);
	}
}
```

新建消费者4，绑定路由规则*.com.chen

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
public class MqConsumer4 {
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
		 //创建一个交换机
		 channel.exchangeDeclare(EXCHANGE_NAME,"topic");
		 //绑定队列到交换机
		 channel.queueBind(queueName, EXCHANGE_NAME, "*.com.chen");
	     System.out.println("Consumer4 Waiting Received messages");
	     //DefaultConsumer类实现了Consumer接口，通过传入一个channel，
	     //告诉服务器我们需要哪个channel的消息并监听channel，如果channel中有消息，就会执行回调函数handleDelivery
	     Consumer consumer = new DefaultConsumer(channel) {
	            @Override
	            public void handleDelivery(String consumerTag, Envelope envelope,
	                                       BasicProperties properties, byte[] body)
	                    throws IOException {
	                String message = new String(body, "UTF-8");
	                System.out.println("Consumer4 Received '" + message + "'");
	            }
	        };
	        //自动回复队列应答 -- RabbitMQ中的消息确认机制
	        channel.basicConsume(queueName, true, consumer);
	}
}
```

&ensp;&ensp;&ensp;&ensp;分别启动消费者1234，使其处于监听状态，可以在RabbitMQ控制台看见声明了一个名为'HelloMq'类型为topic的交换机和四个随机队列

![png2]([RabbitMQ]五：Exchange-topic/png2.png)

![png3]([RabbitMQ]五：Exchange-topic/png3.png)

![png4]([RabbitMQ]五：Exchange-topic/png4.png)

![png5]([RabbitMQ]五：Exchange-topic/png5.png)

![png6]([RabbitMQ]五：Exchange-topic/png6.png)

![png7]([RabbitMQ]五：Exchange-topic/png7.png)

![png8]([RabbitMQ]五：Exchange-topic/png8.png)

&ensp;&ensp;&ensp;&ensp;启动生产者，可以看见消费者12收到了两条消息，消费者34收到了一条消息，实际结果与开篇的预期结果吻合。

![png9]([RabbitMQ]五：Exchange-topic/png9.png)

![png10]([RabbitMQ]五：Exchange-topic/png10.png)

![png11]([RabbitMQ]五：Exchange-topic/png11.png)

![png12]([RabbitMQ]五：Exchange-topic/png12.png)

&ensp;&ensp;&ensp;&ensp;至此，topic类型的交换机功能就实现了。

# 三、代码下载

[下载地址](<https://pan.baidu.com/s/1mj2nEly>)