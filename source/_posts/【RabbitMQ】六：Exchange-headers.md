---
layout: posts
title: 【RabbitMQ】六：Exchange-headers
date: 2018-11-21 16:05:16
tags:
- RabbitMQ
categories: 学习笔记
---

# 一、概述

&ensp;&ensp;&ensp;&ensp;前面三篇文章讲述了RabbitMQ 常用的三种Exchange类型，这篇文章学习一下第四种不常用的Exchange类型：Headers这种类型与topic类型类似，只不过不是匹配routingKeys，是匹配AMQP协议中的Header，Header是一个HashTable类型的键值对，而routingKey是String类型的字符串。功能与Topic相同，消息发送者绑定消息的键值对，匹配交换机与队列之间绑定的键值对，匹配规则“x-match”有两种，一种是“any”，只要一组键值对匹配成功即可发送消息到该队列，另一种是“all”，即需要所有键值对都匹配才可以发送消息。

&ensp;&ensp;&ensp;&ensp;大概的场景应用示意图如下，详细说明见示例代码：

![png1]([RabbitMQ]六：Exchange-headers/png1.png)

# 二、源代码

我们先测试any类型的headers，先写生产者代码，相关说明已在注释中标明

```java
package com.cn.chenxyt.mq;
 
import java.io.IOException;
import java.util.Hashtable;
import java.util.Map;
 
import com.rabbitmq.client.AMQP.BasicProperties;  
import com.rabbitmq.client.AMQP.BasicProperties.Builder; 
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.Channel;
 
 
public class MqProducer {
	 public final static String EXCHANGE_NAME="EX_HEADER";
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
		 channel.exchangeDeclare(EXCHANGE_NAME,"headers");
		 //定义发送消息的要绑定的键值对
		 Map<String,Object> headers =  new Hashtable<String, Object>();
		 headers.put("aaa", "111");
		 headers.put("bbb", "222");  
	     Builder properties = new BasicProperties.Builder();  
	     properties.headers(headers);
		 for(int i = 0;i<500;i++){
			 String message = "hello" + (i);
			 //发送消息 绑定header键值对
			 channel.basicPublish(EXCHANGE_NAME,"",properties.build(),message.getBytes());
			 System.out.println("发送消息：" + message);
			 Thread.sleep(2000);
		 }
	}
}
```

 消费者1代码，相关说明已经在注释中标明

```java
package com.cn.chenxyt.mq;
 
import java.io.IOException;
import java.util.Hashtable;
import java.util.Map;
 
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.Consumer;
import com.rabbitmq.client.DefaultConsumer;
import com.rabbitmq.client.Envelope;
import com.rabbitmq.client.AMQP.BasicProperties;
public class MqConsumer1 {
	 public final static String EXCHANGE_NAME="EX_HEADER";
	 public final static String QUEUE_NAME="queue1";
	 
	 
	 public static void main(String[] args) throws IOException, InterruptedException {
		 //创建连接工厂
		 ConnectionFactory factory = new ConnectionFactory();
		 //设置主机
		 factory.setHost("localhost");
		 //创建一个新的连接 即TCP连接
		 Connection connection = factory.newConnection();
		 //创建一个通道
		 Channel channel = connection.createChannel();
		 //声明队列
		 channel.queueDeclare(QUEUE_NAME, true, false, false, null);
		 //创建一个交换机
		 channel.exchangeDeclare(EXCHANGE_NAME,"headers");
		 //定义绑定规则
	     Map<String, Object> headers = new Hashtable<String, Object>(); 
	     //any 匹配任意一组即可 all 全部匹配
	     headers.put("x-match", "any");
	     headers.put("aaa", "111");  
	     headers.put("bbb", "222");
		 //绑定队列到交换机
		 channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, "",headers);
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
	    	     //   int i = 1/0;
	            }
	        };
	        //自动回复队列应答 -- RabbitMQ中的消息确认机制
	        //false 不自动回复应答 
	        channel.basicConsume(QUEUE_NAME,true, consumer);
	}
}
```

&ensp;&ensp;&ensp;&ensp;消费者2代码，与1基本相同，只不过新建个队列和绑定的header，有一点要说明一下，所有新测试的交换机类型，都需要把之前已经存在的同名的交换机或者同名的队列删除，不然的话新建不会生效，即使参数不同

```java
package com.cn.chenxyt.mq;
 
import java.io.IOException;
import java.util.Hashtable;
import java.util.Map;
 
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.Consumer;
import com.rabbitmq.client.DefaultConsumer;
import com.rabbitmq.client.Envelope;
import com.rabbitmq.client.AMQP.BasicProperties;
public class MqConsumer2 {
	 public final static String EXCHANGE_NAME="EX_HEADER";
	 public final static String QUEUE_NAME="queue2";
	 public static void main(String[] args) throws IOException {
		 //创建连接工厂
		 ConnectionFactory factory = new ConnectionFactory();
		 //设置主机
		 factory.setHost("localhost");
		 //创建一个新的连接 即TCP连接
		 Connection connection = factory.newConnection();
		 //创建一个通道
		 Channel channel = connection.createChannel();
		 //声明队列
		 channel.queueDeclare(QUEUE_NAME, true, false, false, null);
		 //创建一个交换机
		 channel.exchangeDeclare(EXCHANGE_NAME,"headers");
		 //定义绑定规则
	     Map<String, Object> headers = new Hashtable<String, Object>(); 
	     //any 匹配任意一组即可 all 全部匹配
	     headers.put("x-match", "any");
	     headers.put("aaa", "111");  
	     headers.put("ccc", "333");
		 //绑定队列到交换机
		 channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, "",headers);
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
	        channel.basicConsume(QUEUE_NAME, true, consumer);
	}
}
```

&ensp;&ensp;&ensp;&ensp;分别启动消费者1和消费者2，使他们处在监听状态，可以看到管理台有新建的headers类型的交换机和两个队列

![png2]([RabbitMQ]六：Exchange-headers/png2.png)

![png3]([RabbitMQ]六：Exchange-headers/png3.png)

![png4]([RabbitMQ]六：Exchange-headers/png4.png)

![png5]([RabbitMQ]六：Exchange-headers/png5.png)

![png6]([RabbitMQ]六：Exchange-headers/png6.png)

&ensp;&ensp;&ensp;&ensp;启动生产者，可以看到消费者1和消费者2都收到了消息。说明any功能生效，即匹配到了任意一组键值对即可发送消息。

![png7]([RabbitMQ]六：Exchange-headers/png7.png)

![png8]([RabbitMQ]六：Exchange-headers/png8.png)

![png9]([RabbitMQ]六：Exchange-headers/png9.png)

&ensp;&ensp;&ensp;&ensp;接下来我们测试all的情况，在管理台删除已经存在的两条队列queue1、queue2，修改消费者1和消费者2中的any修改为all

```java
	     //any 匹配任意一组即可 all 全部匹配
	     headers.put("x-match", "all");
```

这时我们重新启动消费者1和消费者2使他们处于监听状态，可以在管理台看见绑定规则x-match变为all

![png10]([RabbitMQ]六：Exchange-headers/png10.png)

启动生产者，可以看见消费者1和消费者2都收不到消息了

![png11]([RabbitMQ]六：Exchange-headers/png11.png)

![png12]([RabbitMQ]六：Exchange-headers/png12.png)

![png13]([RabbitMQ]六：Exchange-headers/png13.png)

修改生产者代码，新增一组键值对，保证与queue1绑定的headers键值对完全匹配

```java
		 //定义发送消息的要绑定的键值对
		 Map<String,Object> headers =  new Hashtable<String, Object>();
	     headers.put("aaa", "111");  
	     headers.put("bbb", "222");

```

重新启动生产者，可以看到消费者1收到了消息，消费者2没有收到消息，即all的匹配规则生效了。

![png14]([RabbitMQ]六：Exchange-headers/png14.png)

![png15]([RabbitMQ]六：Exchange-headers/png15.png)

![png16]([RabbitMQ]六：Exchange-headers/png16.png)

&ensp;&ensp;&ensp;&ensp;以上就是关于headers类型的exchange的应用示例，实际应用场景中，同类型的更偏向于使用direct类型的交换机。

# 三、代码下载

[下载地址](<https://pan.baidu.com/s/1jJ5FOQq>)