---
layout: posts
title: 【RabbitMQ】一：基本概念与环境搭建
date: 2018-11-21 09:38:05
tags:
- RabbitMQ
categories: 学习笔记
---

# 一、定义

&ensp;&ensp;&ensp;&ensp;MQ（Message Queue，消息队列）是基于应用程序之间的一种通信方式，应用程序通过读写出入队列的消息进行通信，而不需要用专用的连接来连接它们。消息通信指的是程序之间在消息中传递信息进行通信，而不是传统的通过直接调用（如RPC）的方式进行通信。MQ的使用除去了消息接收和发送的应用程序同时执行的要求。RabbitMQ是实现了AMQP协议MQ的一个中间件，它由ERLANG语言编写。常用的MQ中间件还有IBM MQ，Rocket MQ，Active MQ,Rocket MQ以及轻量级的由Redis实现的MQ。进程之间常用的通信方式有：管道、有名管道、信号量、信号、共享内存、套接字。

# 二、应用场景

&ensp;&ensp;&ensp;&ensp;对于大型的软件系统来说，它会有很多的模块或者说是子系统，比如我在的国内某互联网银行，银行的软件架构体系中就包含了多个系统。这个时候多个系统之间的通信就成了一个问题。传统的IPC（进程间通信）很多都是在单一的系统上，模块之间具有高度的耦合性，不适合做扩展。而使用socket进行通信的话，虽然可以部署在多个服务器上，但是仍然存在许多问题需要解决。诸如：
&ensp;&ensp;&ensp;&ensp;1.信息的发送者与接收者之间如何维持双方之间的连接，如果连接发生中断，发送中的数据如何处理？
&ensp;&ensp;&ensp;&ensp;2.如何降低双方系统之间的耦合度？
&ensp;&ensp;&ensp;&ensp;3.如何能够按照优先级处理数据？
&ensp;&ensp;&ensp;&ensp;4.怎样有效的处理接收者的负载？
&ensp;&ensp;&ensp;&ensp;5.如何使用Filter有效的处理消息到不同的接收者端？
&ensp;&ensp;&ensp;&ensp;6.如何做到可扩展，有效的将消息发送到集群上？
&ensp;&ensp;&ensp;&ensp;7.如何保证消息的可靠性？即接收者能够接收到准确完整的消息。
AMQP协议解决了以上问题，而RabbitMQ实现了AMQP协议

# 三、应用架构

![png1]([RabbitMQ]一：基本概念与环境搭建/png1.png)

&ensp;&ensp;&ensp;&ensp;RabbitMq Server：也叫broker server ，是基于消息发送与接收双方的一种传输服务。他的任务就是维护一条从消息生产者到消息接收者之间的路线，并保证消息能够在其中稳定准确的进行传输。但是这个保证并不能100%保证，不过对于中小型应用系统已经足够了。当然如果对于大型的商业系统，则可以再封装一层数据一致性的guard，就可以彻底保证数据的一致性了。

&ensp;&ensp;&ensp;&ensp;CilentA&ClientB：也叫Producer，消息的发送方，生产者。生产者发送的消息有两个部分：payload和label。payload就是要传递的消息内容，label是exchange的名字或者说是一个tag，RabbitMq 正是通过这个label决定把这个消息发送给哪个接收者。AMQP仅仅描述了label的规则，而RabbitMq决定如何使用这个规则。

&ensp;&ensp;&ensp;&ensp;Client1&Client2：也叫Consumer，消息的接收方，消费者。消息从生产者发送到消费者的时候，label已经被删掉了，也就是说消费者实际上并不知道这个消息是从哪个生产者发来的，除非是payload消息载体中记录了生产者的信息。

&ensp;&ensp;&ensp;&ensp;Exchange：交换机，收发消息的地方，一般情况下消息是直接从生产者发送到队列当中，但是有的时候我们并不知道消息应该发送到哪个队列，这个时候我们就把消息发送到交换机，由交换机根据路由规则决定应该发送到哪个队列中。

&ensp;&ensp;&ensp;&ensp;Queue：队列，存储消息的地方，消费者从队列中获取消息。

&ensp;&ensp;&ensp;&ensp;RoutingKey：路由关键字，决定Exchange到Queue的规则，exchange与queue通过绑定（binding）而实现消息传递。

&ensp;&ensp;&ensp;&ensp;Virtual Hosts：虚拟主机，每一个Virtual Host都相当于一个独立的RabbitMq Server，拥有自己的exchange、queue等，彼此相互独立。这保证了多个应用使用同一个服务器上的RabbitMq Server。

&ensp;&ensp;&ensp;&ensp;Connection：连接，Producer和Consumer通过一条TCP连接建立与RabbitMq Server之间的连接。也就是说，消息在传递之前，第一步就是建立这个TCP连接。

&ensp;&ensp;&ensp;&ensp;Channel：频道，基于TCP连接的通道，数据的传递都是在Channel上进行的。也就是说当程序建立完TCP连接之后，接下来就是建立Channel通道。

&ensp;&ensp;&ensp;&ensp;之所以不直接使用TCP连接传递消息而是使用通道，是因为TCP连接的开启与关闭的代价太大，并且TCP有连接数的限制，限制了系统处理高并发的能力，后边我们会说到，当消费者开启多线程模式时，实际是开启了多个通道，而不是多个TCP连接。

# 四、Exchange类型

&ensp;&ensp;&ensp;&ensp;这里着重说明一下Exchange，因为我在学习跟使用的过程中发现，exchange真的非常的重要，可以说是RabbtiMq中的一个小的核心了。

&ensp;&ensp;&ensp;&ensp;从上边的图中我们可以看到，发送方将消息发送到Exchange中，接着通过绑定的“Routing Key”决定消息应该发送到哪个Queue中。这里有四种Exchange介绍，每种实现绑定了不同的路由规则，即不同的“Routing Key”类型。

四种Exchange的类型分别为：fanout、direct、topic、header

**&ensp;&ensp;&ensp;&ensp;Fanout Exchange：**此种类型的Exchange不需要绑定任何的路由规则，即消息会发送给所有与其绑定的Queue，如图我们定义了fanout类型的交换机X，那么由生产者P发送到交换机X的消息会发送给队列queue1跟队列queue2两个队列

![png2]([RabbitMQ]一：基本概念与环境搭建/png2.png)

**&ensp;&ensp;&ensp;&ensp;Direct Exchange：**此种类型的交换机会匹配绑定的路由规则决定消息由交换机发送给哪个队列。生产者发送给交换机的消息携带绑定的规则，交换机根据规则匹配队列的绑定规则决定消息发送给哪个队列。如图，我们定义了direct类型的交换机X，交换机X与队列queue1的绑定规则为error和warning，与队列queue2的绑定规则为error、info和warning，那么当生产者发送的消息携带绑定规则参数info时，交换机X只会将该消息发送到队列queue2上，当传递的规则参数为error、warning时，交换机X会将该消息发送到队列queue1与队列queue2两个队列

![png3]([RabbitMQ]一：基本概念与环境搭建/png3.png)

&ensp;&ensp;&ensp;&ensp;**Topic Exchange：**此种类型的交换机实际上与direct类型的相似，不同的是它会对绑定规则进行模糊匹配，匹配模式有两种，一种是‘#’匹配一个或多个单词，另一种是‘*’匹配一个单词。如图，我们定义了topic类型的交换机，绑定队列queue1的规则为#，绑定队列queue2的规则为c.*，绑定队列queue3的规则为*.c，那么不管生产者发送绑定什么规则参数的消息，都会被队列queue1收到，当消息的绑定规则参数为c.x，x为任意一个单词时，队列queue2会收到该消息，为x.c时，x为任意单词时，队列queue3会收到该消息

![png4]([RabbitMQ]一：基本概念与环境搭建/png4.png)

&ensp;&ensp;&ensp;&ensp;**Header Exchange:** 此种类型的交换机与direct类型的相似，不同的是不在使用Routing Key进行匹配，而是使用header进行匹配，header是队列绑定时的一个参数。因为没有看到太多使用此种类型的例子，所以此处不过多阐明

# 五、环境搭建

&ensp;&ensp;&ensp;&ensp;RabbitMq 是由erlang语言开发的，所以需要安装erlang语言的OTP支持软件，可以在官网上下载，也可以在我的百度云进行下载。

[下载地址](<https://pan.baidu.com/s/1bpSDLkr>)

&ensp;&ensp;&ensp;&ensp;安装完OTP之后，安装RabbitMq Server，同样可以在官网下载，也可以在我的百度云进行下载。

[下载地址](<https://pan.baidu.com/s/1jJdUOE6>)

&ensp;&ensp;&ensp;&ensp;两个软件的安装过程都是下一步，需要注意的是两个软件安装完成之后都需要配置环境变量，安装完成之后进入RabbitMq Server安装目录下的sbin目录，打开CMD窗口执行命令：rabbitmq-plugins enable rabbitmq_management 以启动监听服务。启动成功之后在浏览器中输入http：localhost:15672 出现如下窗口即表示安装成功。

![png5]([RabbitMQ]一：基本概念与环境搭建/png5.png)

初始用户名密码均为guest

以上就是RabbitMq的一些基础内容，还在学习当中，如果有错误的地方请谅解并欢迎指正。

本文参考文章：

[文章地址](http://blog.csdn.net/anzhsoft/article/details/19563091)

