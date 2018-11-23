---
layout: posts
title: 【RabbitMQ】八：RabbitMQ的消息确认
date: 2018-11-22 10:01:38
tags:
- RabbitMQ
categories: 学习笔记
---

# 一、概述

&ensp;&ensp;&ensp;&ensp;前文说到RabbitMQ的交换机、队列、消息的持久化并不能100%的保证消息不会丢失。首先从生产者端，持久化的消息在RabbitMQ同步到磁盘之前，还需要一段时间，这个时间很短，但是不容忽视。假如此时服务器宕机了，那么消息就丢失了。这种发生在生产者上的消息丢失我们可以使用镜像队列和事务机制来保证数据的完整性。其次是消费者端，假如消费者拿到消息还未处理，发生异常而崩溃，此时这条消息队列中已经没有了，而我们的业务还需要这条消息，那么这种情况也算是消息丢失。在消费者端发生的消息丢失可以通过消费者的消息确认机制来解决。当然无论哪种方式对RabbitMQ的性能都有一定的影响。本文主要对RabbitMQ对于生产者和消费者不同的消息确认方式做一个了解，并解决在消息确认中出现的阻塞问题。

# 二、事务管理（生产者）

&ensp;&ensp;&ensp;&ensp;事务管理的操作是针对于生产者向RabbitMQ服务器发送消息这一过程的。RabbitMQ对事务的管理有如下两个层面的方式：

1、AMQP协议层面，基于AMQP的事务机制

2、通道层面，将channel设置成confirm

## 2.1事务机制

&ensp;&ensp;&ensp;&ensp;RabbitMQ提供了txSelect()、txCommit()和txRollback()三个方法对消息发送进行事务管理，txSelect用于将通道channel开启事务模式，txCommit用于提交事务，txRollback用户进行事务回滚操作。

```java
try{
   //channel开启事务模式
   channel.txSelect();
   //发送消息
   channel.basicPublish(EXCHANGE_NAME,"",MessageProperties.PERSISTENT_TEXT_PLAIN,message.getBytes());
   //模拟异常
   int n = 1/0;
   //提交事务
   channel.txCommit();
}catch(Exception e){
   e.printStackTrace();
   channel.txRollback();
}
```

假如在txCommit之前发生了异常，那么就可以通过Rollback进行回滚操作。

&ensp;&ensp;&ensp;&ensp;以上是基于AMQP协议层的事务机制，确保了数据在生产者与RabbitMQ服务器之间的可靠性，但是性能开销较大。

## 2.2Confirm模式

&ensp;&ensp;&ensp;&ensp;RabbitMQ提供了一种低消耗的事务管理方式，将channel设置成confirm模式。confirm模式的channel，通过该channel发出的消息会生成一个唯一的有序ID（从1开始），一旦消息成功发送到相应的队列之后，RabbitMQ服务端会发送给生产者一个确认标志，包含消息的ID，这样生产者就知道该消息已经发送成功了。如果消息和队列是持久化的，那么当消息成功写入磁盘之后，生产者会收到确认消息。此外服务端也可以设置basic.ack的mutiple域，表明是否是批量确认的消息，即该序号之前的所有消息都已经收到了。

&ensp;&ensp;&ensp;&ensp;confirm的机制是异步的，生产者可以在等待的同时继续发送下一条消息，并且异步等待回调处理，如果消息成功发送，会返回ack消息供异步处理，如果消息发送失败发生异常，也会返回nack消息。confirm的时间没有明确说明，并且同一个消息只会被confirm一次。

&ensp;&ensp;&ensp;&ensp;我们在生产者使用如下代码开启channel的confirm模式，并且已经开启事务机制的channel是不能开启confirm模式的

```java
channel.confirmSelect();
```

处理ack或者nack的方式有三种：

1、串行confirm：每发送一条消息就调用waitForConfirms（）方法等待服务端confirm

```java
//开启confirm模式
channel.confirmSelect();
String message = "Hello World";
//发送消息
channel.basicPublish(EXCHANGE_NAME,"",MessageProperties.PERSISTENT_TEXT_PLAIN,message.getBytes());
//判断是否回复
if(channel.waitForConfirms()){
    System.out.println("Message send success."); 
 }
```

其中waitForConfirms可以换成带有时间参数的方法waitForConfirms(Long mills)指定等待响应时间

2、批量confirm：每发送一批次消息就调用waitForConfirms（）方法等待服务端confirm

```java
//开启confirm模式
channel.confirmSelect();
for(int i =0;i<1000;i++){
    String message = "Hello World";
    //发送消息
    channel.basicPublish(EXCHANGE_NAME,"",MessageProperties.PERSISTENT_TEXT_PLAIN,message.getBytes());
    if(i%100==0){
        //每发送100条判断一次是否回复
        if(channel.waitForConfirms()){
              System.out.println("Message send success."); 
          }
     }
}
```

&ensp;&ensp;&ensp;&ensp;批量的方法从数量级上降低了confirm的性能消耗，提高了效率，但是有个致命的缺陷，一旦回复确认失败，当前确认批次的消息会全部重新发送，导致消息重复发送。所以批量的confirm虽然性能提高了，但是消息的重复率也提高了。

3、异步confirm：使用监听方法，当服务端confirm了一条或多条消息后，调用回调方法

```java
//声明一个用来记录消息唯一ID的有序集合SortedSet
final SortedSet<Long> confirmSet = Collections.synchronizedSortedSet(new TreeSet<Long>());
//开启confirm模式
channel.confirmSelect();
//异步监听方法 处理ack与nack方法
channel.addConfirmListener(new ConfirmListener() {
    //处理ack multiple 是否批量 如果是批量 则将比该条小的所有数据都移除 否则只移除该条
    public void handleAck(long deliveryTag, boolean multiple) throws IOException {
        if (multiple) {
            confirmSet.headSet(deliveryTag + 1).clear();
        } else {
            confirmSet.remove(deliveryTag);
        }
    }
    //处理nack 与ack相同
    public void handleNack(long deliveryTag, boolean multiple) throws IOException {
        System.out.println("There is Nack, SeqNo: " + deliveryTag + ", multiple: " + multiple);
        if (multiple) {
            confirmSet.headSet(deliveryTag + 1).clear();
        } else {
            confirmSet.remove(deliveryTag);
        }
    }
});
while (true) {
    //获取消息confirm的唯一ID
    long nextSeqNo = channel.getNextPublishSeqNo();
    String message = "Hello World.";
    //发送消息
    channel.basicPublish(EXCHANGE_NAME,"",MessageProperties.PERSISTENT_TEXT_PLAIN,message.getBytes());
    //将ID加入到有序集合中
    confirmSet.add(nextSeqNo);
}
```

# 三、消息确认ack（消费者）

&ensp;&ensp;&ensp;&ensp;为了保证RabbitMQ能够感知消费者正确取到了消息，RabbitMQ提供了消息确认机制，与给生产者回复ACK的方式类似，当队列发送一条消息给消费者时，会记录一个unack标志，当消费者拿到消息之后，会回复一个ack标志，从而抵消了原来的unack标志。一般情况下，我们默认是开启了自动回复ack的标志，即当消费者拿到消息之后立即回复ack而不管消息是否正确被处理，这个时间很快，以至于基本看不到unack的状态。如开篇说到，这里存在一个严重的问题，假如消息在业务处理的过程中发生异常crash了，那么这条消息就消失了，持久化也不会解决这个问题。这里就需要我们在日常的业务处理中，消费者要手动的确认消息。确认消息包括两种，一种是ack，另一种是unack，unack是表明我这条消息处理异常了，可以设置参数告诉MQ服务器是否需要将消息重新放入到队列中。同时，如果开启了手动回复确认的消费者，当消费者异常断开时，没有回复的消息会被重新放入队列供给其他消费者使用。所以程序员必须一定要记得回复消息确认，不然会导致消息重复或者大量的消息堆积。

&ensp;&ensp;&ensp;&ensp;下面将通过一个简单的示例，演示手动回复消息确认和忘记回复消息确认的场景。示例场景：一个队列下有两个手动回复消息确认的消费者，两个消费者会按照系统自带的轮训机制获取消息，即一个获取奇数的消息，一个获取偶数的消息。

&ensp;&ensp;&ensp;&ensp;1、消费者1和2手动回复消息（正常情况）

&ensp;&ensp;&ensp;&ensp;2、消费者1和2手动回复消息，且消费者1忘了手动回复并且读取一部分数据之后发生异常（异常情况）

编写生产者代码，生产者发送1000条消息，并且没有消息间隔。

```java
package com.cn.chenxyt.mq;
 
import java.io.IOException;
import java.util.Collections;
import java.util.Hashtable;
import java.util.Map;
import java.util.SortedSet;
import java.util.TreeSet;
 
import com.rabbitmq.client.AMQP.BasicProperties;  
import com.rabbitmq.client.AMQP.BasicProperties.Builder; 
import com.rabbitmq.client.AMQP.Confirm.SelectOk;
import com.rabbitmq.client.ConfirmListener;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.MessageProperties;
 
 
public class MqProducer {
	 public final static String EXCHANGE_NAME="EXCHANGE_MQ";
	 public final static String QUEUE_NAME="queue";
	 public void sendMessage() throws IOException, InterruptedException {
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
		// channel.exchangeDeclare(EXCHANGE_NAME,"fanout");
		 channel.exchangeDeclare(EXCHANGE_NAME, "fanout",true);
		 //创建一个队列
		 channel.queueDeclare(QUEUE_NAME, true, false, false, null);
		 channel.queueBind(QUEUE_NAME, EXCHANGE_NAME,"");
		 for(int i =1;i<1000;i++){
			 String message = "Hello World" + (i);
			 //发送消息
			 channel.basicPublish(EXCHANGE_NAME,"",MessageProperties.PERSISTENT_TEXT_PLAIN,message.getBytes());
			 System.out.println("Message send success:" + message); 
		 }
 
	}
}
```

接下来编写消费者代码，与之前相同，有两个消费者，消费者1和消费者2

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
	 private static String EXCHANGE_NAME="EXCHANGE_MQ";
	 private final static String QUEUE_NAME="queue";
	 public static void main(String[] args) throws IOException {
		 //创建连接工厂
		 ConnectionFactory factory = new ConnectionFactory();
		 //设置主机
		 factory.setHost("localhost");
		 //创建一个新的连接 即TCP连接
		 Connection connection = factory.newConnection();
		 //创建一个通道
		 final Channel channel = connection.createChannel();
		 //声明队列
		 channel.queueDeclare(QUEUE_NAME, true, false, false, null);
		 //创建一个交换机
		 channel.exchangeDeclare(EXCHANGE_NAME,"fanout",true);		 
		 //绑定队列到交换机
		 channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, "",null);
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
	                try {
						Thread.sleep(2000);
					} catch (InterruptedException e) {
						// TODO Auto-generated catch block
						e.printStackTrace();
					}finally{
							channel.basicAck(envelope.getDeliveryTag(), false);
					}
	            }
	        };
	        //自动回复队列应答 -- RabbitMQ中的消息确认机制
	        //false 不自动回复应答 
	        channel.basicConsume(QUEUE_NAME,false, consumer);
	}
}
```

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
	 private static String EXCHANGE_NAME="EXCHANGE_MQ";
	 private final static String QUEUE_NAME="queue";
	 public static void main(String[] args) throws IOException {
		 //创建连接工厂
		 ConnectionFactory factory = new ConnectionFactory();
		 //设置主机
		 factory.setHost("localhost");
		 //创建一个新的连接 即TCP连接
		 Connection connection = factory.newConnection();
		 //创建一个通道
		 final Channel channel = connection.createChannel();
		 //声明队列
		 channel.queueDeclare(QUEUE_NAME, true, false, false, null);
		 //创建一个交换机
		 channel.exchangeDeclare(EXCHANGE_NAME,"fanout",true);		 
		 //绑定队列到交换机
		 channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, "",null);
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
	                try {
						Thread.sleep(2000);
					} catch (InterruptedException e) {
						// TODO Auto-generated catch block
						e.printStackTrace();
					}finally{
							channel.basicAck(envelope.getDeliveryTag(), false);
					}
	            }
	        };
	        //自动回复队列应答 -- RabbitMQ中的消息确认机制
	        //false 不自动回复应答 
	        channel.basicConsume(QUEUE_NAME,false, consumer);
	}
}
```

&ensp;&ensp;&ensp;&ensp;这里basicConsume设置为false为不自动应答，同时为了保证业务正常执行完，回复确认要写在finally代码块里。channel.basicAck（）回复处理正确，channel.basicNAck（）回复处理失败，参数设置为true为重新加入队列。

启动消费者1和2再启动生产者，因为两个消费者对消息延迟2s才回复，所以队列中积累了大量的unack消息

![png1]([RabbitMQ]八：RabbitMQ的消息确认/png1.png)

![png2]([RabbitMQ]八：RabbitMQ的消息确认/png2.png)

![png3]([RabbitMQ]八：RabbitMQ的消息确认/png3.png)

&ensp;&ensp;&ensp;&ensp;接下来修改消费者1代码，看一下如果程序没有回复ack确认是什么样子，注释掉消费者1的ack确认，并把生产者发送数据条数改成10条（这里如果在上边的例子改，需要保证队列里没有数据，可以在管理台把队列删掉，也可以停掉消费者把sleep时间改短然后启动把之前的消息接收完毕，当然也可以在上边测试的时候就把发送消息的数目改小一些）

```java
     try {
						Thread.sleep(2000);
					} catch (InterruptedException e) {
						// TODO Auto-generated catch block
						e.printStackTrace();
					}finally{
							//channel.basicAck(envelope.getDeliveryTag(), false);
					}
```

&ensp;&ensp;&ensp;&ensp;接下来启动消费者1和消费者2以及生产者，可以看到10条消息，有五条发送到消费者1，五条发送到消费者2，同时在消息接收完毕的时候，由于消费者1没有ack，所以管理台上一直有5个unack状态

![png4]([RabbitMQ]八：RabbitMQ的消息确认/png4.png)

![png5]([RabbitMQ]八：RabbitMQ的消息确认/png5.png)

![png6]([RabbitMQ]八：RabbitMQ的消息确认/png6.png)

&ensp;&ensp;&ensp;&ensp;这时我们停掉消费者1，模拟消费者1crash断开的状态，可以看到消费者2收到了消费者1没有ack的消息，并且管理台队列里的unack状态也没有了

![png7]([RabbitMQ]八：RabbitMQ的消息确认/png7.png)

![png8]([RabbitMQ]八：RabbitMQ的消息确认/png8.png)

以上就是关于消费者自动回复消息确认的相关内容。

# 四、阻塞的问题解决

&ensp;&ensp;&ensp;&ensp;这里思考一个问题，就是当消费者1和消费者2都开启手动回复并且在业务执行完成之后都进行了回复，如果生产者发送了大量消息，而消费者处理业务的时间（我们用sleep时间模拟）又过长，就会导致消息队列中阻塞大量未unack的消息，会降低系统性能，即便我们把消费者2的sleep时间调低，消费者1仍然是2s处理一条消息，消费者2迅速处理完，队列中仍然积累一半unack的消息，这是为什么呢？这是因为每个消费者会有一个缓冲池prefetch的概念，prefetch是消费者一次能处理的最大unack的数量，消费者获取消息时，实际上是mq先放到了这个缓冲池中，当ack一个之后，mq从缓冲池中拿掉一个。而MQ的轮训机制恰好是按顺序分发，因为我们这里没有设置缓冲池的大小，也就是消费者一次最多能拿多少个消息没有设置，所以MQ默认你的处理能力很好，会按照顺序将消息全部分发完。所以这里就会看到消费者1刚好打印的都是奇数的消息，消费者2刚好打印的是偶数的消息。

&ensp;&ensp;&ensp;&ensp;所以阻塞的问题的解决方案就是我们合理的设置prefetch大小，这样处理快的消费者就能够处理更多的消息，处理慢的消费者也不会发生长时间的阻塞。更详细的描述，假设有两个消费者，都设置prefetch大小为10，消费者1处理业务时间是2s，消费者2处理业务时间是2ms，那么就不会出现上边的情况消费者1积累大量的unack，这里最多的unack数目就是两个prefetch的大小之和20，同时，MQ分发消息是先塞满10个到消费者1，再塞满10个到消费者2，塞第21个的时候，先看消费者1的缓冲池有没有空位，没有的话去看消费者2，因为消费者2的处理速度比1快1000倍，所以1000条数据前10条塞给消费者1之后，后边的数据就都塞给消费者2了。

设置prefetch大小的方法，在消费者中加入如下代码

```java
		 channel.basicQos(10);
```

&ensp;&ensp;&ensp;&ensp;为了更好的说明上边的详细描述，我把代码贴出来，变化就是生产者一次发1000条信息，消费者1和消费者2设置最大prefetch值为10，同时消费者1的处理业务时间（sleep时间）2s，消费者2处理业务时间2ms

```java
package com.cn.chenxyt.mq;
 
import java.io.IOException;
import java.util.Collections;
import java.util.Hashtable;
import java.util.Map;
import java.util.SortedSet;
import java.util.TreeSet;
 
import com.rabbitmq.client.AMQP.BasicProperties;  
import com.rabbitmq.client.AMQP.BasicProperties.Builder; 
import com.rabbitmq.client.AMQP.Confirm.SelectOk;
import com.rabbitmq.client.ConfirmListener;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.MessageProperties;
 
 
public class MqProducer {
	 private final static String EXCHANGE_NAME="EXCHANGE_MQ";
	 private final static String QUEUE_NAME="queue";
	 
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
		// channel.exchangeDeclare(EXCHANGE_NAME,"fanout");
		 channel.exchangeDeclare(EXCHANGE_NAME, "fanout",true);
		 //创建一个队列
		 channel.queueDeclare(QUEUE_NAME, true, false, false, null);
		 channel.queueBind(QUEUE_NAME, EXCHANGE_NAME,"");
		 for(int i =1;i<=1000;i++){
			 String message = "Hello World" + (i);
			 //发送消息
			 channel.basicPublish(EXCHANGE_NAME,"",MessageProperties.PERSISTENT_TEXT_PLAIN,message.getBytes());
			 System.out.println("Message send success:" + message); 
		 }
 
	}
}
```

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
	 private static String EXCHANGE_NAME="EXCHANGE_MQ";
	 private final static String QUEUE_NAME="queue";
	 public static void main(String[] args) throws IOException {
		 //创建连接工厂
		 ConnectionFactory factory = new ConnectionFactory();
		 //设置主机
		 factory.setHost("localhost");
		 //创建一个新的连接 即TCP连接
		 Connection connection = factory.newConnection();
		 //创建一个通道
		 final Channel channel = connection.createChannel();
		 //声明队列
		 channel.queueDeclare(QUEUE_NAME, true, false, false, null);
		 //创建一个交换机
		 channel.exchangeDeclare(EXCHANGE_NAME,"fanout",true);		 
		 //绑定队列到交换机
		 channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, "",null);
		 channel.basicQos(10);
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
	                try {
						Thread.sleep(2000);
					} catch (InterruptedException e) {
						// TODO Auto-generated catch block
						e.printStackTrace();
					}finally{
							//channel.basicAck(envelope.getDeliveryTag(), false);
					}
	            }
	        };
	        //自动回复队列应答 -- RabbitMQ中的消息确认机制
	        //false 不自动回复应答 
	        channel.basicConsume(QUEUE_NAME,false, consumer);
	}
}
```

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
	 private static String EXCHANGE_NAME="EXCHANGE_MQ";
	 private final static String QUEUE_NAME="queue";
	 public static void main(String[] args) throws IOException {
		 //创建连接工厂
		 ConnectionFactory factory = new ConnectionFactory();
		 //设置主机
		 factory.setHost("localhost");
		 //创建一个新的连接 即TCP连接
		 Connection connection = factory.newConnection();
		 //创建一个通道
		 final Channel channel = connection.createChannel();
		 //声明队列
		 channel.queueDeclare(QUEUE_NAME, true, false, false, null);
		 //创建一个交换机
		 channel.exchangeDeclare(EXCHANGE_NAME,"fanout",true);		 
		 //绑定队列到交换机
		 channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, "",null);
		 channel.basicQos(10);
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
	                try {
						Thread.sleep(2);
					} catch (InterruptedException e) {
						// TODO Auto-generated catch block
						e.printStackTrace();
					}finally{
							channel.basicAck(envelope.getDeliveryTag(), false);
					}
	            }
	        };
	        //自动回复队列应答 -- RabbitMQ中的消息确认机制
	        //false 不自动回复应答 
	        channel.basicConsume(QUEUE_NAME,false, consumer);
	}
}
```

&ensp;&ensp;&ensp;&ensp;这里为了验证unak的数目与prefetch的关系，我们消费者1注掉回复确认消息的代码，启动消费者1和消费者2以及生产者

![png9]([RabbitMQ]八：RabbitMQ的消息确认/png9.png)

![png10]([RabbitMQ]八：RabbitMQ的消息确认/png10.png)

![png11]([RabbitMQ]八：RabbitMQ的消息确认/png11.png)

如图可见，消费者1只处理了10条消息，消费者2把其他的消息处理了。合理的利用了有限的资源。

# 五、总结

&ensp;&ensp;&ensp;&ensp;本文主要讲述了RabbitMQ与生产者和消费者之间的消息确认以及消费者手动确认消息带来的阻塞问题的解决之道。生产者的消息确认有事务机制和confirm模式两种，消费者通过自动回复ack和手动回复ack的方式确认，手动ack切记有ack和nack两种，合理安排使用。消费者手动确认带来的阻塞问题是由于没有设置缓冲池的大小，可以通过设置prefetch的大小来限制每个消费者能持有的最大unack的数量，合理的分配资源