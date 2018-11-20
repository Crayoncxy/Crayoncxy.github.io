---
layout: posts
title: 【Websocket】Blocking message pending 10000 for BLOCKING
date: 2018-11-20 15:13:55
tags:
- Websocket
categories: 问题解决
---

# 一、问题描述：

&ensp;&ensp;&ensp;&ensp;最近学习了一下Websocket，然后使用它结合spring做了一个实施监控的页面（使用的是spring-websocket），功能是监控交易发生，当交易发生时主动推送给前端页面展示交易流水。测试的时候一个线程顺序批量执行接口没有什么问题，页面哗啦啦的显示很顺利。当启动两个线程以上之后跑了一会儿之后就宕掉了。日志报错“Blocking message pending 10000 for BLOCKING”
&ensp;&ensp;&ensp;&ensp;这里我本地写了一个测试Demo模拟了一下报错的场景，主要代码如下：
在WebSocketHandler中的一个静态方法，用来给前端推送消息的方法：

```java
    public static void sendMessageToUser(String userName, TextMessage message) {
        for (WebSocketSession user : users) {
            if (user.getAttributes().get("WEBSOCKET_USERNAME").equals(userName)) {
                try {
                    if (user.isOpen()) {
                    	user.sendMessage(message);
                    }
                } catch (IOException e) {
                    e.printStackTrace();
                }
                break;
            }
        }
    }
```

在WebSocketHandler中的监听方法，当前端建立起与服务端的Websocket通信之后触发：

```java
    public void afterConnectionEstablished(WebSocketSession session) throws Exception {
        // TODO Auto-generated method stub
    	users.add(session);
        System.out.println("connect to the websocket success......当前数量:"+users.size());
        System.out.println("userName ::" + session.getAttributes().get("WEBSOCKET_USERNAME")) ;
        Thread thread1 = new TestThread(session);
        thread1.start();
        while(true){
			Random rand = new Random();
			int i = rand.nextInt(10000);
			TextMessage imessage = new TextMessage(String.valueOf(i));
        	sendMessageToUser((String)session.getAttributes().get("WEBSOCKET_USERNAME"),imessage);
        }
    }
```

&ensp;&ensp;&ensp;&ensp;上边的方法建立连接之后先启动一个子线程调用发消息的方法给前端推送消息，然后主线程也给前端推送消息。子线程如下：

```java
package com.dcits.api.rateStatistics.websocket;
import java.util.Random;
import org.springframework.web.socket.TextMessage;
import org.springframework.web.socket.WebSocketSession;
 
public class TestThread extends Thread{
	public WebSocketSession session;
	
	public TestThread(WebSocketSession session) {
		// TODO Auto-generated constructor stub
		this.session = session;
	}
	@Override
	public void run() {
		while(true){
			Random rand = new Random();
			int i = rand.nextInt(10000);
			TextMessage imessage = new TextMessage(String.valueOf(i));
			SpringWebSocketHandler.sendMessageToUser((String)session.getAttributes().get("WEBSOCKET_USERNAME"),imessage);
		}
	}
}
```

运行报错信息：

```java
Exception in thread "Thread-25" java.lang.IllegalStateException: Blocking message pending 10000 for BLOCKING
	at org.eclipse.jetty.websocket.common.WebSocketRemoteEndpoint.lockMsg(WebSocketRemoteEndpoint.java:130)
	at org.eclipse.jetty.websocket.common.WebSocketRemoteEndpoint.sendString(WebSocketRemoteEndpoint.java:379)
	at org.springframework.web.socket.adapter.jetty.JettyWebSocketSession.sendTextMessage(JettyWebSocketSession.java:184)
	at org.springframework.web.socket.adapter.AbstractWebSocketSession.sendMessage(AbstractWebSocketSession.java:104)
	at com.dcits.api.rateStatistics.websocket.SpringWebSocketHandler.sendMessageToUser(SpringWebSocketHandler.java:85)
	at com.dcits.api.rateStatistics.websocket.TestThread.run(TestThread.java:23)
```

# 二、解决方案：

&ensp;&ensp;&ensp;&ensp;网上查了一圈，基本都是StackOverFlow上的一些英文问答

![png1]([Websocket]Blocking-message-pending-10000-for-BLOCKING/png1.png)

唯一一条中文问答还答非所问

![png2]([Websocket]Blocking-message-pending-10000-for-BLOCKING/png2.png)

&ensp;&ensp;&ensp;&ensp;大概看了一下其中一个StackOverFlow的答案，原文部分如下:

```java
I don't thing you necessarily have to use Futures or mechanisms described in the above post. What I don't really get is : why doing asynchronous call to servlets ? Ofcourse several could send messages on the same RemoteEndPoint.. But can't you simply make synchronous calls to the relevant classes and keep the same request-response flow that you use when records are found in your database ? :)
```

&ensp;&ensp;&ensp;&ensp;我理解是这样的，在使用同一个RemoteEndPoint节点发送消息时，发生了阻塞。而阻塞的原因是因为异步调用，即一条消息没有发送给前端呢就已经开始准备发送第二条消息了，然后就产生了阻塞。有人提供了一种解决方案是使用Future多线程模型，将要处理的消息发送方法放在FutureTask中执行，等待正确返回之后再执行下一次发送。我这里的话，使用Future未免有点小题大做了，因为实际上我不需要等待异步执行的结果，也就是消息只要发出去就好。因为我使用synchronized关键字把sendMessageToUser方法加了锁，从而保证同一时间只发送一条消息。此时此刻我开了10个线程在跑交易，页面正在有条不紊的哗哗哗刷新着。后边如果再出现问题再想办法解决吧。