---
layout: posts
title: >-
  【Zookeeper】Unable to read additional data from client sessionid*,likely client
  has closed socket
date: 2018-11-20 14:25:26
tags:
- Zookeeper
categories: 问题解决
---

# 一、问题描述：

&ensp;&ensp;&ensp;&ensp;因为项目中使用到了Zookeeper，所以我自己找了些关于zk的资料学习了一下。在异步创建节点的过程中，抛出了如下问题：

![png1]([Zookeeper]Unable to read additional data from client sessionid , likely client has closed socket/png1.png)

&ensp;&ensp;&ensp;&ensp;异步创建节点的时候总是闪退，然后服务端报错 Unable to read additional data from client sessionid 0x162b246bfe50000, likely client has closed socket ，我们先看下代码 这里我把同步跟异步的代码一起贴了出来便于学习

```java
package zk.zkTest;
import java.io.IOException;
import java.util.Random;
 
import org.apache.zookeeper.AsyncCallback.DataCallback;
import org.apache.zookeeper.AsyncCallback.StringCallback;
import org.apache.zookeeper.CreateMode;
import org.apache.zookeeper.KeeperException;
import org.apache.zookeeper.KeeperException.Code;
import org.apache.zookeeper.WatchedEvent;
import org.apache.zookeeper.Watcher;
import org.apache.zookeeper.ZooDefs;
import org.apache.zookeeper.ZooKeeper;
import org.apache.zookeeper.KeeperException.NoNodeException;
import org.apache.zookeeper.KeeperException.NodeExistsException;
import org.apache.zookeeper.data.Stat;
//实现一个监视点Watcher接口
public class Master implements Watcher{
	
	public static final String HOST_POT = "127.0.0.1:2181";
	
	//zk连接对象
	public ZooKeeper zk;
	//连接地址、端口
	public String hostpot;
	//构造函数 传入连接参数
	Master(String hostpot){
		this.hostpot = hostpot;
	}
	//启动
	void zkStart(){
		try {
			//this传入该类的对象
			zk = new ZooKeeper(hostpot,400000,this);
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}
	//关闭
	void zkStop(){
		try {
			zk.close();
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}
	//创建主节点存储的数据 
	Random random = new Random();
	String serverId = Integer.toHexString(random.nextInt());
	//创建主节点（同步）
	void runForMasterSyn() throws InterruptedException{
		//循环的原因是 当客户端与服务端发生网络中断时 客户端会尝试重新连接服务端
		while(true){
			try {
				// 同步调用方法 第一个参数节点名称 第二个是节点数据 第三个是安全策略 此处使用开放式的ACL访问控制列表 第四个是创建模式 此处创建了一个临时节点
				zk.create("/master",serverId.getBytes(),ZooDefs.Ids.OPEN_ACL_UNSAFE,CreateMode.EPHEMERAL);
				isLeader = true;
				break;
			} catch (NodeExistsException e) {
				// TODO Auto-generated catch block  
				e.printStackTrace();
				isLeader = false;
				break;
			} catch (KeeperException e) {
				// TODO Auto-generated catch block 网络连接异常 此处不确定主节点是否已经创建完成 网络断开可能发生在请求之前，也可能发生在请求创建之后 因此要考虑这种异常 所以此处不返回 执行下面的check判断
				e.printStackTrace();
			}
			if(checkMasterSyn()){
				break;
			}
		}
	}
		
	public static boolean isLeader = false;
	//检查主节点是否已经创建 发生上述两种异常时执行 （同步）
	boolean checkMasterSyn() throws InterruptedException{
		while(true){
			Stat stat = new Stat();
			try {
				//获取节点的数据 第二个字段为是否启动监听后续的变化
				byte data[] = zk.getData("/master",false,stat);
				//如果数据里的内容是当前的进程存储的内容 则表示当前进程为主节点 即创建成功
				isLeader = new String(data).equals(serverId);
				return true;
			} catch (NoNodeException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
				return false;
			} catch (KeeperException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}		
	}
	
	//创建主节点（异步） 与同步相比多了两个参数 1.回调方法 2.指定的上下文信息 回调方法调用的是传入的对象的实例 （回调要知道是哪个调用的回调）同时因为回调返回之前不需要等待create结果 所以没有Interrupt异常 因为执行结果会在回调之后收到 所以也没有Keeper异常
	void runForMasterAsyn(){
		zk.create("/master",serverId.getBytes(),ZooDefs.Ids.OPEN_ACL_UNSAFE,CreateMode.EPHEMERAL,masterCreateCallBack,null);
	}
	//创建回调对象 异步调用只有一个线程处理回调请求 因此遵循FIFO的原则 所以应避免异步调用里处理大量逻辑或者阻塞代码
	StringCallback masterCreateCallBack = new StringCallback(){
		//回调的函数  rc 返回调用的结果 返回OK或者异常编码 path 对应create的第一个参数 ctx 对应create的上下文 即最后一个字段  name znode节点名  对于非有序节点 path与name相同 有序节点 name会自动标号
		public void processResult(int rc, String path, Object ctx,
				String name) {
			// TODO Auto-generated method stub
			switch (Code.get(rc)){
			case CONNECTIONLOSS:
				checkMasterAsyn();
				return;
			case OK:
				isLeader = true;
				break;
			default:
				isLeader = false;
			}
			System.out.println("I'm "+(isLeader?"":"not")+" Leader");
			
		}
	};
	//检查主节点状态 （异步调用创建主节点时做的检查）
	void checkMasterAsyn(){
		//第一个参数节点名称 第二个参数是否设置监听 第三个为获取数据的异步回调 第四个为上下文信息
		zk.getData("/master",false,masterCheckCallBack,null);
	}
	//异步获取节点数据信息
	DataCallback masterCheckCallBack = new DataCallback(){
		public void processResult(int rc, String path, Object ctx,
				byte[] data, Stat stat) {
			// TODO Auto-generated method stub
			switch(Code.get(rc)){
			case CONNECTIONLOSS:
				//此处递归循环调用 尝试连接
				checkMasterAsyn();
				return;
			case NONODE:
				//没有节点重新创建
				runForMasterAsyn();
				return;
			}
		}
		
	};
	//实现的接口方法 
	public void process(WatchedEvent arg0) {
		// TODO Auto-generated method stub
		System.out.println(arg0);
	}
	public static void main(String[] args) throws InterruptedException {
		Master m = new Master(HOST_POT);
		m.zkStart();
		m.runForMasterAsyn();
		if(isLeader){
			//此处用来处理主节点业务逻辑
			System.out.println("I'M THE LEADER");
			Thread.sleep(2000000000);
		}else{
			System.out.println("SORRY,SOMEONE ELSE IS THE LEADER");
		}
		m.zkStop();
	}
}
```

我们看下运行之后的控制台信息：

![png2]([Zookeeper]Unable to read additional data from client sessionid , likely client has closed socket/png2.png)

# 二、解决方案：

&ensp;&ensp;&ensp;&ensp;从上边控制台的信息可以看出 if/else的判断发生在了异步回调之前，并且打印了最后一行日志Session closed
所以这里抛错的原因是：**没有等待异步通知的响应信息，就提前关闭了连接。**解决方法就是在if/else之前延时等待，或者设置变量等待异步通知返回结果之后再进行if/else判断。

&ensp;&ensp;&ensp;&ensp;网上还有很多关于这个错误的解决方案，场景不同，但大多数的原因都是因为网络中断，有的可能是超时时间不够，我这里的原因是在异步通知返回结果之前就人为的结束了连接。