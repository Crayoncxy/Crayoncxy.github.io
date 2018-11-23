---
layout: posts
title: 【RabbitMQ】十：RabbitMQ与Spring 的整合
date: 2018-11-23 09:38:22
tags:
- RabbitMQ
categories: 学习笔记
---

# 一、概述

&ensp;&ensp;&ensp;&ensp;前面的文章中整理了常规项目下RabbitMQ实现各种通用消息队列的方式，一般的企业级项目，通常使用Spring框架来实现项目，本文主要讲述RabbitMQ与Spring的集成，通过一个简单的示例演示集成。

&ensp;&ensp;&ensp;&ensp;示例：通过Spring管理项目，实现RabbitMQ的fanout类型交换机的消息队列，一个生产者Producer，一个fanout类型的交换机exchangeTest，两个队列queueTest和queueTest1以及两个消费者Consumer和Consumer1接收消息

![png1]([RabbitMQ]十：RabbitMQ与Spring 的整合/png1.png)

# 二、代码

首先是在pom文件中加入用于整合的依赖

```xml
		<!--rabbitmq依赖 -->
		<dependency>
			<groupId>org.springframework.amqp</groupId>
			<artifactId>spring-rabbit</artifactId>
			<version>1.3.5.RELEASE</version>
		</dependency
```

同时贴出完整的pom依赖

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
 
  <groupId>smq</groupId>
  <artifactId>smqp</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>jar</packaging>
	<properties>
		<!-- spring版本号 -->
		<spring.version>3.2.8.RELEASE</spring.version>
		<!-- log4j日志文件管理包版本 -->
		<slf4j.version>1.6.6</slf4j.version>
		<log4j.version>1.2.12</log4j.version>
		<!-- junit版本号 -->
		<junit.version>4.10</junit.version>
	</properties>
 
	<dependencies>
		<!-- 添加Spring依赖 -->
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-core</artifactId>
			<version>${spring.version}</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-webmvc</artifactId>
			<version>${spring.version}</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-context</artifactId>
			<version>${spring.version}</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-context-support</artifactId>
			<version>${spring.version}</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-aop</artifactId>
			<version>${spring.version}</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-aspects</artifactId>
			<version>${spring.version}</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-tx</artifactId>
			<version>${spring.version}</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-jdbc</artifactId>
			<version>${spring.version}</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-web</artifactId>
			<version>${spring.version}</version>
		</dependency>
 
		<!--单元测试依赖 -->
		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<version>${junit.version}</version>
			<scope>test</scope>
		</dependency>
 
		<!-- 日志文件管理包 -->
		<!-- log start -->
		<dependency>
			<groupId>log4j</groupId>
			<artifactId>log4j</artifactId>
			<version>${log4j.version}</version>
		</dependency>
		<dependency>
			<groupId>org.slf4j</groupId>
			<artifactId>slf4j-api</artifactId>
			<version>${slf4j.version}</version>
		</dependency>
		<dependency>
			<groupId>org.slf4j</groupId>
			<artifactId>slf4j-log4j12</artifactId>
			<version>${slf4j.version}</version>
		</dependency>
		<!-- log end -->
 
		<!--spring单元测试依赖 -->
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-test</artifactId>
			<version>${spring.version}</version>
			<scope>test</scope>
		</dependency>
 
		<!--rabbitmq依赖 -->
		<dependency>
			<groupId>org.springframework.amqp</groupId>
			<artifactId>spring-rabbit</artifactId>
			<version>1.3.5.RELEASE</version>
		</dependency>
 
		<dependency>
			<groupId>javax.validation</groupId>
			<artifactId>validation-api</artifactId>
			<version>1.1.0.Final</version>
		</dependency>
 
		<dependency>
			<groupId>org.hibernate</groupId>
			<artifactId>hibernate-validator</artifactId>
			<version>5.0.1.Final</version>
		</dependency>
 
	</dependencies>
	<build>
		<resources>
			<resource>
				<directory>src/main/resources</directory>
				<targetPath>${basedir}/target/classes</targetPath>
				<includes>
					<include>**/*.properties</include>
					<include>**/*.xml</include>
				</includes>
				<filtering>true</filtering>
			</resource>
			<resource>
				<directory>src/main/resources</directory>
				<targetPath>${basedir}/target/resources</targetPath>
				<includes>
					<include>**/*.properties</include>
					<include>**/*.xml</include>
				</includes>
				<filtering>true</filtering>
			</resource>
		</resources>
 
		<plugins>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-compiler-plugin</artifactId>
				<configuration>
					<source>1.6</source>
					<target>1.6</target>
					<encoding>UTF-8</encoding>
				</configuration>
			</plugin>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-war-plugin</artifactId>
				<version>2.1.1</version>
				<configuration>
					<warSourceExcludes>${warExcludes}</warSourceExcludes>
				</configuration>
			</plugin>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-surefire-plugin</artifactId>
				<version>2.4.3</version>
				<configuration>
					<testFailureIgnore>true</testFailureIgnore>
				</configuration>
			</plugin>
			<plugin>
				<inherited>true</inherited>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-source-plugin</artifactId>
				<executions>
					<execution>
						<id>attach-sources</id>
						<goals>
							<goal>jar</goal>
						</goals>
					</execution>
				</executions>
			</plugin>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-resources-plugin</artifactId>
				<configuration>
					<encoding>UTF-8</encoding>
				</configuration>
			</plugin>
		</plugins>
	</build>
</project>
```

&ensp;&ensp;&ensp;&ensp;然后新建rabbitMQ.xml文件，该文件是用来管理RabbitMQ的连接以及交换机、队列、消息监听等配置，相关说明已经在注释中描述

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:rabbit="http://www.springframework.org/schema/rabbit"
 xsi:schemaLocation="http://www.springframework.org/schema/beans
     http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
     http://www.springframework.org/schema/beans
     http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
     http://www.springframework.org/schema/rabbit
     http://www.springframework.org/schema/rabbit/spring-rabbit-1.0.xsd">
 <!--配置connection-factory，指定连接rabbit server参数 -->
 <rabbit:connection-factory id="connectionFactory"
  username="guest" password="guest" host="127.0.0.1" port="5672"/>
 <!--定义rabbit template用于数据的接收和发送 -->
 <rabbit:template id="amqpTemplate"  connection-factory="connectionFactory" 
     exchange="exchangeTest" />
 <!--通过指定下面的admin信息，当前producer中的exchange和queue会在rabbitmq服务器上自动生成 -->
 <rabbit:admin connection-factory="connectionFactory" />
 <!--定义queue -->
 <rabbit:queue name="queueTest" durable="true" auto-delete="false" exclusive="false" />
 <!-- 将queue与exchange进行fanout绑定 -->
 <rabbit:fanout-exchange name="exchangeTest" durable="true" auto-delete="false">
  <rabbit:bindings>
   <rabbit:binding queue="queueTest"></rabbit:binding>
  </rabbit:bindings>
 </rabbit:fanout-exchange> 
  <!--定义queue1 -->
 <rabbit:queue name="queueTest1" durable="true" auto-delete="false" exclusive="false" />
 <!-- 将queue1与exchange进行fanout绑定 -->
 <rabbit:fanout-exchange name="exchangeTest" durable="true" auto-delete="false">
  <rabbit:bindings>
   <rabbit:binding queue="queueTest1"></rabbit:binding>
  </rabbit:bindings>
 </rabbit:fanout-exchange> 
 <!-- 消息接收者 -->
 <bean id="messageReceiver1" class="com.cn.chenxyt.consumer.MessageConsumer1"></bean>
 <!-- queue litener  观察 监听模式 当有消息到达时会通知监听在对应的队列上的监听对象-->
    <rabbit:listener-container connection-factory="connectionFactory" concurrency="20" prefetch="1">
             <rabbit:listener queues="queueTest" ref="messageReceiver1"/>
    </rabbit:listener-container>
    <!-- 消息接收者2 -->
 <bean id="messageReceiver2" class="com.cn.chenxyt.consumer.MessageConsumer2"></bean>
 <!-- queue1 litener  观察 监听模式 当有消息到达时会通知监听在对应的队列上的监听对象-->
    <rabbit:listener-container connection-factory="connectionFactory">
             <rabbit:listener queues="queueTest1" ref="messageReceiver2"/>
    </rabbit:listener-container>
</beans>
```

在Spring的配置文件application.xml指定扫描包的路径，将注释注册为Spring Beans

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
	xmlns:p="http://www.springframework.org/schema/p"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.1.xsd
	http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-3.1.xsd">
	<import resource="classpath*:rabbitmq.xml" />
	<!-- 扫描指定package下所有带有如@controller,@services,@resource,@ods并把所注释的注册为Spring Beans -->
	<context:component-scan base-package="com.cn.chenxyt.producer,com.cn.chenxyt.consumer" />
	<!-- 激活annotation功能 -->
	<context:annotation-config />
	<!-- 激活annotation功能 -->
	<context:spring-configured />
</beans>
```

创建消息生产者 MessageProducer.java 通过在配置文件管理的bean来管理消息的发送

```java
package com.cn.chenxyt.producer;
import javax.annotation.Resource;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.amqp.core.AmqpTemplate;
import org.springframework.stereotype.Service;
 
@Service
public class MessageProducer {
	private Logger logger = LoggerFactory.getLogger(MessageProducer.class);
	@Resource
	private AmqpTemplate amqpTemplate;
	public void sendMessage(Object message){
	  logger.info("我要发送消息给消费者:{}",message);
	  amqpTemplate.convertAndSend("这条消息来自生产者==" +message);
	}
}
```

创建消息消费者1和消费者2，建立监听，监听名与配置文件中的监听相同

```java
package com.cn.chenxyt.consumer;
 
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.amqp.core.Message;
import org.springframework.amqp.core.MessageListener;
 
public class MessageConsumer1 implements MessageListener {
	private Logger logger = LoggerFactory.getLogger(MessageConsumer1.class);
	@Override
	public void onMessage(Message message) {
		logger.info("我是消费者1我收到消息了:{}",message);
	}
}
```

```java
package com.cn.chenxyt.consumer;
 
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.amqp.core.Message;
import org.springframework.amqp.core.MessageListener;
 
public class MessageConsumer2 implements MessageListener {
	private Logger logger = LoggerFactory.getLogger(MessageConsumer2.class);
	@Override
	public void onMessage(Message message) {
		logger.info("我是消费者2我收到消息了:{}",message);
	}
}
```

创建启动类，加载spring配置文件，并获取发送者的bean进行消息发送

```java
package com.cn.chenxyt.producer;
import java.util.Random;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
public class ProjectStart {
	private static ApplicationContext context = null;
	public static void main(String[] args) throws InterruptedException {
		context = new ClassPathXmlApplicationContext("application.xml");
		MessageProducer messageProducer = (MessageProducer) context.getBean("messageProducer");
		Random random = new Random(); 
		while(true){
			Thread.sleep(2000);
			messageProducer.sendMessage(random.nextInt());	
		}
	}
 
}
```

启动ProjectStart.java可以看见控制台打印出消息的内容

![png2]([RabbitMQ]十：RabbitMQ与Spring 的整合/png2.png)

以上就是RabbitMQ整合Spring的示例.

# 三、关于缓冲池以及并发的控制

&ensp;&ensp;&ensp;&ensp;前文讲过关于RabbitMQ消息处理缓冲池prefetch的概念，当两个消费者处理消息的能力差距在千倍级别以上时，可以考虑通过改变prefetch大小的方式来合理的利用资源。这里在整合了Spring之后，还可以通过给处理慢的消费者增开线程的方式来提高处理速度（查阅了很多资料，没有找到如何在不整合Spring的前提下增加线程数量）。对比上文rabbitMQ.xml文件中，两个消费者监听的配置是有不一样的地方的

消费者1

```xml
	<!-- 消息接收者 -->
	<bean id="messageReceiver1" class="com.cn.chenxyt.consumer.MessageConsumer1"></bean>
	<!-- queue litener  观察 监听模式 当有消息到达时会通知监听在对应的队列上的监听对象-->
    <rabbit:listener-container connection-factory="connectionFactory" concurrency="20" prefetch="1">
             <rabbit:listener queues="queueTest" ref="messageReceiver1"/>
    </rabbit:listener-container>

```

消费者2

```xml
    <!-- 消息接收者2 -->
	<bean id="messageReceiver2" class="com.cn.chenxyt.consumer.MessageConsumer2"></bean>
	<!-- queue1 litener  观察 监听模式 当有消息到达时会通知监听在对应的队列上的监听对象-->
    <rabbit:listener-container connection-factory="connectionFactory">
             <rabbit:listener queues="queueTest1" ref="messageReceiver2"/>
    </rabbit:listener-container>
```

&ensp;&ensp;&ensp;&ensp;可以看到这里同样有prefetch参数并且功能与之前说的相同，[[RabbitMQ学习笔记八：RabbitMQ的消息确认](http://blog.csdn.net/chenxyt/article/details/79259838)]一文中关于阻塞问题的解决。此外这里消费者1还多了个concurrency参数，这个参数是当前开启的线程总数，也就是同时处理消息的线程数，一个线程启动一个channel通道。这里prefetch =1 concurrency=20与单纯的只设置prefetch=20是不同的，前者启动了20个线程，假设消费者处理能力缓慢，但是也会同时处理20个消息，后者只是单纯的表示我同一时间可以从队列中拿到多少消息，如果消费者处理能力缓慢，需要按顺序执行消息。也就是说前者的效率要远大于后者。

&ensp;&ensp;&ensp;&ensp;这里我们模拟演示不同场景下分别使用concurrency和不使用的情况

&ensp;&ensp;&ensp;&ensp;场景1：生产者1s发送一条消息，发送20条，消费者 20s处理一条，不使用concurrency以及prefetch

修改配置文件：

```xm
	<!-- 消息接收者 -->
	<bean id="messageReceiver1" class="com.cn.chenxyt.consumer.MessageConsumer1"></bean>
	<!-- queue litener  观察 监听模式 当有消息到达时会通知监听在对应的队列上的监听对象-->
    <rabbit:listener-container connection-factory="connectionFactory">
             <rabbit:listener queues="queueTest" ref="messageReceiver1"/>
    </rabbit:listener-container>
```

修改生产者代码，间隔1s发送一条消息，发送20条

```java
package com.cn.chenxyt.producer;
import java.util.Random;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
public class ProjectStart {
	private static ApplicationContext context = null;
	public static void main(String[] args) throws InterruptedException {
		context = new ClassPathXmlApplicationContext("application.xml");
		MessageProducer messageProducer = (MessageProducer) context.getBean("messageProducer");
		Random random = new Random(); 
		for(int i =0;i<20;i++){
			Thread.sleep(1000);
			messageProducer.sendMessage(random.nextInt());	
		}
	}
 
}
```

修改消费者1代码，模拟任务处理时间20s，同时防止日志干扰，注释掉消费者2打印日志的代码

```java
package com.cn.chenxyt.consumer;
 
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.amqp.core.Message;
import org.springframework.amqp.core.MessageListener;
 
public class MessageConsumer1 implements MessageListener {
	private Logger logger = LoggerFactory.getLogger(MessageConsumer1.class);
	@Override
	public void onMessage(Message message) {
		try {
			Thread.sleep(20000);
			logger.info("消费者1线程:{}",message);
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}
}
```

&ensp;&ensp;&ensp;&ensp;这里有个地方要注意，每做一个测试之前，要先看RabbitMQ控制台已经存在的队列中是否有未处理的消息，有的话及时处理或删除，以免影响测试结果。

&ensp;&ensp;&ensp;&ensp;启动ProjectStart。控制台间隔1s打印一条发送消息的日志，一共打印20条，并且间隔20s会打印一条收到消息的日志。RabbitMQ管理台上可以看到开始的时候有1个处于unacked状态的消息，然后从1递增到19个ready状态的消息，total从1递增到20之后开始每隔20s减1，直到最后全部为0（ready是队列中存在的未被消费者接收的消息数量，unacked是被消费者接收但是未返回的消息数量，total是他们二者之和），可以看到eclipse的控制台和RabbitMQ的管理台展示的结果相同，均表明此种情况是一种阻塞状况，20条消息是按照顺序执行的，全部执行完的时间大约是20x20=400s

![png3]([RabbitMQ]十：RabbitMQ与Spring 的整合/png3.png)

![png4]([RabbitMQ]十：RabbitMQ与Spring 的整合/png4.png)

&ensp;&ensp;&ensp;&ensp;场景2：生产者1s发送1条消息，发送20条，消费者20s处理一条消息，设置消费者的concurrency=20，prefetch=1，即启动20个线程，每个线程能缓冲的最大消息数目为1

修改配置文件

```xml
    <rabbit:listener-container connection-factory="connectionFactory" concurrency="20" prefetch="1">
             <rabbit:listener queues="queueTest" ref="messageReceiver1"/>
    </rabbit:listener-container>
```

&ensp;&ensp;&ensp;&ensp;等待上一个测试的消息完全处理完成之后，启动ProjectStart.java 可以看到控制台依然间隔1s打印一条发送消息，同时在20s之后开始间隔1s打印一条收到消息，并且RabbitMQ控制台ready，unacked，total数目与eclipse控制台状态相同，同时我们可以看到启动了20个线程，即创建了20个通道。测试结果表明，消息发送出去的时候，就已经被消费者接收了，只不过消息间隔1s发送，所以消息也是间隔1s接收，然后延迟20s打印，所以这种情况处理速度约为20+20=40s

![png5]([RabbitMQ]十：RabbitMQ与Spring 的整合/png5.png)

![png6]([RabbitMQ]十：RabbitMQ与Spring 的整合/png6.png)

![png7]([RabbitMQ]十：RabbitMQ与Spring 的整合/png7.png)

&ensp;&ensp;&ensp;&ensp;场景3：生产者间隔1s发送一条消息，发送20次，消费者间隔20s处理一条消息，设置消费者prefetch=20

修改配置文件：

```xml
    <rabbit:listener-container connection-factory="connectionFactory" prefetch="20">
             <rabbit:listener queues="queueTest" ref="messageReceiver1"/>
    </rabbit:listener-container>
```

&ensp;&ensp;&ensp;&ensp;确保上次测试的消息成功处理完之后启动ProjectStart.java，控制台间隔1s打印一条消息发出日志，间隔20s打印一条消息收到日志，与场景1的结果相同，不同的地方在于，RabbitMQ控制台ready状态为0，unacked状态间隔1s递增到20之后间隔20s递减，这种情况说明，消息都被消费者拿走了，但是由于消费者处理能力有限（一个线程，间隔时间20s）所以虽然一次拿了20个消息，但是仍然是顺序执行，20s处理一条数据。与场景1不同的是，场景1压力主要在RabbitMQ服务端，而该场景压力在消费者上。这种情况的处理速度与场景1相同约为20x20=400s

![png8]([RabbitMQ]十：RabbitMQ与Spring 的整合/png8.png)

![png9]([RabbitMQ]十：RabbitMQ与Spring 的整合/png9.png)

以上就是关于整合了spring之后的RabbitMQ并发测试相关内容。

# 四、代码下载

[下载地址](<https://pan.baidu.com/s/1pNoRXNl>)