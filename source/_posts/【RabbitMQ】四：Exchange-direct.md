---
layout: posts
title: 【RabbitMQ】四：Exchange-direct
date: 2018-11-21 15:28:50
tags:
- RabbitMQ
categories: 学习笔记
---

# 一、概述

&ensp;&ensp;&ensp;&ensp;前一篇文章中，我们了解了交换机的四种类型，并使用了一个广播式发送消息的示例学习了fanout类型的交换机，这篇文章继续学习direct类型的交换机。direct类型的交换机可以实现不同的消息发送到不同的队列中去。

&ensp;&ensp;&ensp;&ensp;示例场景：在绑定同一个交换机的两个队列中，一个队列负责接收生产者发送的奇数消息，一个队列负责接收生产者发送的偶数消息，当发送的消息为10的整数倍时，两个队列均可收到消息。（目的是为了验证direct类型的交换机可以同时实现fanout功能）

&ensp;&ensp;&ensp;&ensp;实现方式：使用direct类型的交换机，分别定义绑定在交换机上的两个队列不同的routingKey，和一个相同的routingKey，发送消息指定routingKey，交换机根据指定的routingKey路由到指定的队列中。

![png1]([RabbitMQ]四：Exchange-direct/png1.png)

# 二、源代码

&ensp;&ensp;&ensp;&ensp;整个工程的代码与上一篇的代码相似，只不过是改变了交换机的类型，同时一个消费者队列绑定了两个routingKey

&ensp;&ensp;&ensp;&ensp;先写生产者代码，相关描述都写在注释中，当是奇数时，发到绑定为odd的队列，是偶数时发到绑定为even的队列，是10的倍数时发送到绑定为all的队列。

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
		 channel.exchangeDeclare(EXCHANGE_NAME,"direct");
		 for(int i =1;i<1000;i++){
			 if(i%10 == 0){
				 //10的倍数
				 String message = "我是10的倍数===》" + i;
				 channel.basicPublish(EXCHANGE_NAME,"all",null,message.getBytes("UTF-8"));
				 System.out.println("发给所有队列===" + i);
			 }else{
				 if(i%2 != 0){
					 //发送奇数
					 String message = "我是奇数===》" + i;
					 channel.basicPublish(EXCHANGE_NAME,"odd",null,message.getBytes("UTF-8"));
					 System.out.println("发给队列1===" + i);
				 }else{
					 //发送偶数
					 String message = "我是偶数===》" + i;
					 channel.basicPublish(EXCHANGE_NAME,"even",null,message.getBytes("UTF-8"));
					 System.out.println("发给队列2===" + i);
				 }
			 }
			 Thread.sleep(2000);
		 }
	}
}
```

再写消费者1代码，同样注释有说明，同时，随机队列绑定了odd和all两种routingKey

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
		 channel.queueBind(queueName, EXCHANGE_NAME, "odd");
		 channel.queueBind(queueName, EXCHANGE_NAME, "all");
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

再写消费者2代码，与消费者1相同，随机队列绑定了even和all两种routingKey

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
		 channel.queueBind(queueName, EXCHANGE_NAME, "even");
		 channel.queueBind(queueName, EXCHANGE_NAME, "all");
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

然后分别启动消费者1和消费者2，使两个队列处于监听状态。

![png2]([RabbitMQ]四：Exchange-direct/png2.png)

![png3]([RabbitMQ]四：Exchange-direct/png3.png)

&ensp;&ensp;&ensp;&ensp;再启动生产者，可以看见RabbitMQ 控制台创建了一个名为“HelloMq”类型为direct的交换机，同时可以看见生产者发送数字，消费者1接收到奇数，消费者2接收到偶数，当数字为10的倍数时，消费者1和2都接收到消息。

![png4]([RabbitMQ]四：Exchange-direct/png4.png)

![png5]([RabbitMQ]四：Exchange-direct/png5.png)

![png6]([RabbitMQ]四：Exchange-direct/png6.png)

![png7]([RabbitMQ]四：Exchange-direct/png7.png)

&ensp;&ensp;&ensp;&ensp;至此，一个direct类型的交换机功能就实现了。工作中的应用场景可以系统中做日志处理，两个队列分别打印debug与error级别的日志，同时打印info级别的日志。

# 三、代码下载

[下载地址](<https://pan.baidu.com/s/1kXnCIvd>)