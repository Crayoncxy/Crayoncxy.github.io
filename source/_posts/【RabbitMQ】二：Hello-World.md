---
layout: posts
title: 【RabbitMQ】二：Hello World
date: 2018-11-21 10:10:24
tags:
- RabbitMQ
categories: 学习笔记
---

# 一、流程概述

&ensp;&ensp;&ensp;&ensp;本文主要实现RabbitMQ在JAVA项目中的入门级别应用，即实现消息生成者发送一条‘’Hello World“ 消息，消费者收到这条信息并打印出来。消息的传递流程是“生产者-队列-消费者”，没有经过交换机，如图

![png1]([RabbitMQ]二：Hello-World/png1.png)

**P**：消息生产者

**QUEUE**：队列

**X**：消息接收者

# 二、源代码

&ensp;&ensp;&ensp;&ensp;我这里用的是一个maven项目，首先要在pom.xml中引入RabbitMq依赖的jar包

```html
    <dependency>
    <groupId>com.rabbitmq</groupId>
    <artifactId>amqp-client</artifactId>
    <version>3.0.4</version>
    </dependency>
```

&ensp;&ensp;&ensp;&ensp;然后编写生产者代码，相关内容已经在注释中做了说明，这里说一下端口号的问题，我个人理解15672是服务管理台的端口号，而5672才是RabbitMQ 客户端的端口号。其次是关于队列的创建先后顺序问题，队列在生成者创建和在消费者创建都行，因为在rabbitMQ中，队列是以名字区分的，已经存在的队列再次创建之前不删除是不会生效的，即便是改了参数。

```java
package com.cn.chenxyt.mq;
import java.io.IOException;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.Channel;
 
 
public class MqProducer {
	 public final static String QUEUE_NAME="HelloMq";
	 public static void main(String[] args) throws IOException, InterruptedException {
		 //创建连接工厂
		 ConnectionFactory factory = new ConnectionFactory();
		 //设置主机、用户名、密码和RabbitMQ客户端端口
		 factory.setHost("localhost");
		 factory.setUsername("guest");
		 factory.setPassword("guest");
		 factory.setPort(5672);
		 //创建一个新的连接 即TCP连接
		 Connection connection = factory.newConnection();
		 //创建一个通道
		 Channel channel = connection.createChannel();
		 String message = "Hello World";
		 while (true){
			 //发送消息
			 channel.basicPublish("",QUEUE_NAME,null,message.getBytes("UTF-8"));
	         System.out.println("Producer Send +'" + message + "!");
	         Thread.sleep(2000);
		 }
	}	 
}
```

&ensp;&ensp;&ensp;&ensp;然后我们编写消费者代码，同样相关说明已经写在注释中。这里队列的声明我们是在消费者端，所以启动的时候理应当先启动消费者，再启动生产者。因为生产者写了个死循环，所以先启动生成者再启动消费者，消费者能够收到启动之后的消息，之前的消息这里我还是有些疑问，因为没有队列声明，所以可能是没有发到rabbitMQ。另外，如果连接本地的RabbitMQ，则没有特殊要求的时候用户名、密码、端口都可以不写。

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
public class MqConsumer {
	 public final static String QUEUE_NAME="HelloMq";
	 public static void main(String[] args) throws IOException {
		 //创建连接工厂
		 ConnectionFactory factory = new ConnectionFactory();
		 //设置主机
		 factory.setHost("localhost");
		 //创建一个新的连接 即TCP连接
		 Connection connection = factory.newConnection();
		 //创建一个通道
		 Channel channel = connection.createChannel();
		 //创建队列
	     channel.queueDeclare(QUEUE_NAME, false, false, true, null);
	     System.out.println("Customer Waiting Received messages");
	     //DefaultConsumer类实现了Consumer接口，通过传入一个channel，
	     //告诉服务器我们需要哪个channel的消息并监听channel，如果channel中有消息，就会执行回调函数handleDelivery
	     Consumer consumer = new DefaultConsumer(channel) {
	            @Override
	            public void handleDelivery(String consumerTag, Envelope envelope,
	                                       BasicProperties properties, byte[] body)
	                    throws IOException {
	                String message = new String(body, "UTF-8");
	                System.out.println("Customer Received '" + message + "'");
	            }
	        };
	        //自动回复队列应答 -- RabbitMQ中的消息确认机制
	        channel.basicConsume(QUEUE_NAME, true, consumer);
	}	 
}
```

接下来启动消费者，我们可以看到控制台打出等待消息：

![png2]([RabbitMQ]二：Hello-World/png2.png)

&ensp;&ensp;&ensp;&ensp;并且在RabbitMQ的管理台可以看到建立了一个connection，一个channel和一个名为“HelloMq”的队列，因为是消费者监听的队列，队列中并没有消息流动传输，所以通道与队列的状态都为idle    

![png3]([RabbitMQ]二：Hello-World/png3.png)

![png4]([RabbitMQ]二：Hello-World/png4.png)

![png5]([RabbitMQ]二：Hello-World/png5.png)

接下来我们启动消息生产者，可以看到控制台每隔2s打印一条发送的消息

![png6]([RabbitMQ]二：Hello-World/png6.png)

并且消息消费者的控制台接收到了消息

![png7]([RabbitMQ]二：Hello-World/png7.png)

同时可以在管理台上看见建立了两个连接connection，两个处于活动的channel和一个处于活动的队列

![png8]([RabbitMQ]二：Hello-World/png8.png)

![png9]([RabbitMQ]二：Hello-World/png9.png)

![png10]([RabbitMQ]二：Hello-World/png10.png)

至此，一个使用JAVA语言编写的简单的RabbitMQ实例就完成了。

# 三、代码下载

[下载地址](<https://pan.baidu.com/s/1dOm1Eu>)

初次学习， 若有错误的地方望及时指正，感谢。