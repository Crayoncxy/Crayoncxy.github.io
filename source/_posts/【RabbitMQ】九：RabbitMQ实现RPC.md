---
layout: posts
title: 【RabbitMQ】九：RabbitMQ实现RPC
date: 2018-11-22 10:34:24
tags:
- RabbitMQ
categories: 学习笔记
---

# 一、概述

&ensp;&ensp;&ensp;&ensp;前面几篇文章讲述的内容都是单向的消息传递，生产者将消息发送给消费者之后就不再管后续的业务处理了。实际业务中，有的时候我们还需要等待消费者返回结果给我们，或者是说我们需要消费者上的一个功能、一个方法或是一个接口返回给我们相应的值，而往往大型的系统软件，生产者跟消费者之间都是相互独立的两个系统，部署在两个不同的电脑上，不能通过直接对象.方法的形式获取想要的结果，这时候我们就需要用到RPC（Remote Procedure Call）远程过程调用方式。

&ensp;&ensp;&ensp;&ensp;RabbitMQ实现RPC的方式很简单，生产者发送一条带有标签（消息ID（correlation_id）+回调队列名称）的消息到发送队列，消费者（也称RPC服务端）从发送队列获取消息并处理业务，解析标签的信息将业务结果发送到指定的回调队列，生产者从回调队列中根据标签的信息获取发送消息的返回结果。

![png1]([RabbitMQ]九：RabbitMQ实现RPC/png1.png)

&ensp;&ensp;&ensp;&ensp;如图，客户端C发送消息，指定消息的ID=rpc_id，回调响应的队列名称为rpc_resp，消息从C发送到rpc_request队列，服务端S获取消息业务处理之后，将correlation_id附加到响应的结果发送到指定的回调队列rpc_resp中，客户端从回调队列获取消息，匹配与发送消息的correlation_id相同的值为消息应答结果。

&ensp;&ensp;&ensp;&ensp;RabbitMQ官网的示例是客户端通过RPC方式调用服务端获取斐波那契数列的值，我们举个简单的例子，客户端通过RPC调用获取服务端求平方的方法返回值。即客户端发送消息4，服务端返回4的平方16

# 二、几个简单的概念

## 2.1回调队列

&ensp;&ensp;&ensp;&ensp;前文说到，客户端发送消息到服务端之后，要接收返回结果，存放返回结果的队列叫做回调队列，客户端发送消息之后阻塞监听该队列返回的消息。我们可以使用随机队列命名，也可以指定队列的名称，同时，我们可以一个消息建立一个随机队列，但是通常考虑资源使用的情况，我们一般一个消费者建立一个指定的回调队列。

## 2.2消息属性

&ensp;&ensp;&ensp;&ensp;AMQP协议预定了一组14个消息属性（Message Properties），常用的有如下四种消息属性

    1、deliveryMode：标记消息传递模式，为2时表示持久化消息，其它值不做持久化，在前面文章讲到消息持久化使用的PERSITNAT_TEXT_PLAIN时提到过
    2、contentType：内容类型，用于描述内容编码，如json
    3、replyTo：应答，指定的通用的回调队列名称
    4、correlationId：关联ID，指定消息的标签，方便关联RPC的请求与响应
&ensp;&ensp;&ensp;&ensp;上述四个属性我们使用了replyTo和correlationId属性，同时因为RPC调用是具有幂等性的，所以我们可以忽视不属于我们应该获得到的correlationId。

# 三、示例代码

&ensp;&ensp;&ensp;&ensp;我们梳理一下文章开始提到的简单示例，我们的RPC处理流程如下

    1、客户端启动，创建请求队列rpc_request和回调队列rpc_resp
    2、客户端为我们的消息请求设置两个消息属性correlationId关联ID和replyTo回调队列（rpc_resp）
    3、将请求发送到rpc_request队列
    4、RPC服务端监听rpc_request队列中的请求，获取消息处理业务，并把带有接收消息的correlationId的返回结果消息返回到指定回调队列（rpc_resp）
    5、客户端监听rpc_resp回调队列，如果有消息，匹配correlationId，如果和请求消息的相同，那么这个消息就是返回的响应结果了。
客户端代码MqRpcClient，相关说明已经写在注释中

```java
package com.cn.chenxyt.mq;
 
import java.io.IOException;
import java.util.Collections;
import java.util.Hashtable;
import java.util.Map;
import java.util.Random;
import java.util.SortedSet;
import java.util.TreeSet;
 
import com.rabbitmq.client.AMQP.BasicProperties;  
import com.rabbitmq.client.AMQP.BasicProperties.Builder; 
import com.rabbitmq.client.AMQP.Confirm.SelectOk;
import com.rabbitmq.client.ConfirmListener;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.ConsumerCancelledException;
import com.rabbitmq.client.MessageProperties;
import com.rabbitmq.client.QueueingConsumer;
import com.rabbitmq.client.ShutdownSignalException;
 
 
public class MqRpcClient{
	 private final static String REQUEST_QUEUE_NAME="rpc_request";
	 private final static String RESPONSE_QUEUE_NAME="rpc_resp";
	 private Channel channel;
	 private QueueingConsumer qConsumer;
	 
	 //构造函数 初始化连接
	 public MqRpcClient() throws IOException, InterruptedException {
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
		 channel = connection.createChannel();
		 //创建一个请求队列
		 channel.queueDeclare(REQUEST_QUEUE_NAME, true, false, false, null);
		 //创建一个回调队列
		 channel.queueDeclare(RESPONSE_QUEUE_NAME,true,false,false,null);
		 //为通道创建一个监听（用于监听回调队列，获取返回消息）
		 qConsumer = new QueueingConsumer(channel);
		 //关联监听与监听队列 并手动应答
		 channel.basicConsume(RESPONSE_QUEUE_NAME,false,qConsumer);
	}
	 public String getSquare(String message) throws  Exception{
		 String response = "";
		 //定义消息属性中的correlationId
		 String correlationId = java.util.UUID.randomUUID().toString();
		 //设置消息属性的replTo和correlationId
		 BasicProperties properties = new BasicProperties.Builder().correlationId(correlationId).replyTo(RESPONSE_QUEUE_NAME).build();
		 //发送消息到请求队列rpc_request队列 ，前边说到过 如果没有exchange即没有routingKey 消息发送到与routingKey参数相同的队列中
		 channel.basicPublish("",REQUEST_QUEUE_NAME, properties,message.getBytes());
		 //阻塞监听
		 while(true){
			 QueueingConsumer.Delivery delivery = qConsumer.nextDelivery();
			 if(delivery.getProperties().getCorrelationId().equals(correlationId)){
				 response = new String(delivery.getBody(),"UTF-8");
		    	 //手动回应消息应答
		    	 channel.basicAck(delivery.getEnvelope().getDeliveryTag(),false);
				 break;
			 }
		 }
		 return response;
	 }
	 public static void main(String[] args) throws Exception {
		MqRpcClient rpcClient = new MqRpcClient();
		String result = rpcClient.getSquare("4");
		System.out.println("resonse is :" + result);
	}
}
```

服务端代码MqRpcServer

```java
package com.cn.chenxyt.mq;
import java.io.IOException;
import java.text.NumberFormat;
import java.util.Hashtable;
import java.util.Map;
 
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.Consumer;
import com.rabbitmq.client.ConsumerCancelledException;
import com.rabbitmq.client.DefaultConsumer;
import com.rabbitmq.client.Envelope;
import com.rabbitmq.client.AMQP.BasicProperties;
import com.rabbitmq.client.QueueingConsumer;
import com.rabbitmq.client.ShutdownSignalException;
public class MqRpcServer {
	 private final static String REQUEST_QUEUE_NAME="rpc_request";
	 
	 public static void main(String[] args) throws Exception{
		 //创建连接工厂
		 ConnectionFactory factory = new ConnectionFactory();
		 //设置主机
		 factory.setHost("localhost");
		 //创建一个新的连接 即TCP连接
		 Connection connection = factory.newConnection();
		 //创建一个通道
		 final Channel channel = connection.createChannel();
		 //声明队列
		 channel.queueDeclare(REQUEST_QUEUE_NAME, true, false, false, null);
		 //设置prefetch值 一次处理1条数据
		 channel.basicQos(1);
		 //为请求队列设置监听 监听客户端请求 并手动应答
		 QueueingConsumer qConsumer = new QueueingConsumer(channel);
	     channel.basicConsume(REQUEST_QUEUE_NAME,false, qConsumer);
	     System.out.println("Server waiting Requeust.");
	     while(true){
	    	 QueueingConsumer.Delivery delivery = qConsumer.nextDelivery();
	    	 //将请求中的correlationId设置到回调的消息中
	    	 BasicProperties properties = delivery.getProperties();
	    	 BasicProperties replyProperties = new BasicProperties.Builder().correlationId(properties.getCorrelationId()).build();
	    	 //获取客户端指定的回调队列名
	    	 String replyQueue = properties.getReplyTo();
	    	 //返回获取消息的平方
	    	 String message = new String(delivery.getBody(),"UTF-8");
	    	 System.out.println("waiting message is:" + message);
	    	 Double mSquare =  Math.pow(Integer.parseInt(message),2);
	    	 String repMsg = String.valueOf(mSquare);
	    	 channel.basicPublish("",replyQueue,replyProperties,repMsg.getBytes());
	    	 //手动回应消息应答
	    	 channel.basicAck(delivery.getEnvelope().getDeliveryTag(),false);
	     }
	}
}
```

 启动客户端、启动服务端，可以看到控制台打出的结果，符合我们的预期结果，管理台也新建了两条队列。

![png2]([RabbitMQ]九：RabbitMQ实现RPC/png2.png)

![png3]([RabbitMQ]九：RabbitMQ实现RPC/png3.png)

![png4]([RabbitMQ]九：RabbitMQ实现RPC/png4.png)

# 四、总结

&ensp;&ensp;&ensp;&ensp;综上，RabbitMQ的RPC调用方式就是形成了两条队列，两个客户端（服务端相互监听），需要注意的是，如前篇所述，如果开启手动回复，要记得在代码中手动回复ACK。

# 五、代码下载

[下载地址](<https://pan.baidu.com/s/1pNspa0F>)