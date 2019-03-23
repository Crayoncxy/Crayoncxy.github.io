---
layout: posts
title: 【源码学习】Java并发之CountDownLatch
date: 2019-03-23 09:50:29
tags:
- 源码
- 并发编程  
categories: 源码学习
typora-copy-images-to: 【源码学习】Java并发之CountDownLatch
---

# 一、Demo演示

&ensp;&ensp;&ensp;&ensp;在并发的场景中，常见的一个场景就是一个任务的执行要等待多个线程都执行完毕之后才执行。CountDownLatch可以很好的解决这个问题。

以下以多线程累加为背景，演示如何使用CountDownLatch

首先先看不使用CountDownLatch的时候

```java
package com.chenxyt.threadTest;
public class CountDownLatchDemo {
	
	public static int i = 0;
	public static void main(String[] args) {
		for(int j=0;j<10;j++){
			Thread thread = new Thread(){
				@Override
				public void run() {
					// TODO Auto-generated method stub
					add();
				}
			};
			thread.start();
		}
		System.out.println(i);
	}
	public synchronized static void add(){
		i++;
	}
}
```

运行结果：

![1553308547970](【源码学习】Java并发之CountDownLatch\1553308547970.png)

再一次运行：

![1553308568992](【源码学习】Java并发之CountDownLatch\1553308568992.png)

&ensp;&ensp;&ensp;&ensp;这里我们如果多次运行的话，会产生随机的结果。原因就是线程并不是按照我们代码看到的逻辑顺序执行的。循环创建的十个线程和主线程都有机会得到CPU的调度，当主线程被调度执行print代码时，有可能会有子线程还没有执行。因此我们需要使用一些措施来让主线程等待着子线程执行完再执行print代码。这时候CountDownLatch就登场了。

接下来演示CountDownLatch的使用：

```java
package com.chenxyt.threadTest;
import java.util.concurrent.CountDownLatch;
public class CountDownLatchDemo {
	static int i = 0;
	static CountDownLatch ctl = new CountDownLatch(10);
	public static void main(String[] args) {
		for(int j=0;j<10;j++){
			Thread thread = new Thread(){
				@Override
				public void run() {
					// TODO Auto-generated method stub
					add();
				}
			};
			thread.start();
		}
		try {
			ctl.await();
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		System.out.println(i);
	}
	public synchronized static void add(){
		i++;
		ctl.countDown();
	}
}
```

运行结果：

![1553308877746](【源码学习】Java并发之CountDownLatch\1553308877746.png)

&ensp;&ensp;&ensp;&ensp;这回我们无论运行多少次，最终的结果都是10。我们先初始化了一个CountDownLatch对象，参数为10，然后每个线程执行完自己的任务之后，调用了对象的countDown（）方法，主线程调用了对象的await（）方法，最后就得到了预期的结果。下边分析如何实现的。

# 二、源码分析

这里我们通过分析源码来知道CountDownLatch的工作原理

## 2.1CountDownLatch（int）

CountDownLatch是通过构造器完成初始化的

```java
    public CountDownLatch(int count) {
        if (count < 0) throw new IllegalArgumentException("count < 0");
        this.sync = new Sync(count);
    }
```

&ensp;&ensp;&ensp;&ensp;构造器参数是int类型变量，当入参小于0时会抛出非法参数异常。然后是调用了Sync的构造器，构造了Sync对象，并把入参继续下传。接下来我们看下Sync是什么。

### 2.1.1Sync

```java
    private static final class Sync extends AbstractQueuedSynchronizer {
        private static final long serialVersionUID = 4982264981922014374L;
		//获取共享资源 将共享资源state设置成count
        Sync(int count) {
            setState(count);
        }
		
        //查看当前共享资源个数
        int getCount() {
            return getState();
        }
		
        //重写获取共享资源方法
        protected int tryAcquireShared(int acquires) {
            //如果当前共享资源个数为0则返回1  否则返回-1
            //即获取成功为1 失败为负1
            return (getState() == 0) ? 1 : -1;
        }
		//重写释放共享资源方法
        protected boolean tryReleaseShared(int releases) {
            // Decrement count; signal when transition to zero
            //自旋保证内部要执行的终态一定会成功
            for (;;) {
                //获取当前资源个数
                int c = getState();
                //如果资源为0 则返回释放失败 false
                if (c == 0)
                    return false;
                //资源个数-1
                int nextc = c-1;
                //cas操作将资源个数-1
                if (compareAndSetState(c, nextc))
                    //成功则退出自旋 返回true
                    return nextc == 0;
            }
        }
    }
```

&ensp;&ensp;&ensp;&ensp;通过上边的代码我们可以看到，Sync是CountDownLatch的一个内部类，同时也是AbstractQueuedSynchronizer的一个子类。AQS前文有说过，是一个基于volatile共享变量state和FIFO双循环队列的多线程同步器框架，也就是说，Sync是我们自定义的一个同步器。自定义的同步器是重写了tryAcquire/tryRelease独占式或者是tryAcquireShared/tryReleaseShared共享式中的一组或全部，主要是通过对state的set/get/cas操作完成共享变量的获取和释放。

#### 2.1.1.1Sync（int）

这里先只展开说一下Sync的构造函数，其它的函数下文再说，并且在上述代码中已经写了注释。

```java
        Sync(int count) {
            setState(count);
        }
```

&ensp;&ensp;&ensp;&ensp;如同我们说的自定义同步器的功能一样，通过get/set/cas操作完成对state的修改。这里的构造函数是使用了基类AQS的setState（count）方法，将同步器共享资源的state的值设置成了count。

### 2.1.2小结

&ensp;&ensp;&ensp;&ensp;综上所述，CountDownLatch的构造方法，就是共享式的获取了count个资源。通过内部类Sync继承了AQS并通过构造函数传参修改了共享变量的值。

## 2.2countDown()

接下来我们看一下Demo中每个线程执行完之后调用的countDown（）方法是如何实现的。

```java
    public void countDown() {
        sync.releaseShared(1);
    }
```

&ensp;&ensp;&ensp;&ensp;哦嚯，竟然如此简单。只调用了sync对象的releaseShared方法，并传参数1。内部类Sync的方法在上一节中已经给出，我们可以看到内部类中并没有releaseShared（int）这个方法，因此可以判定这个方法是AQS的。没错，之前学习AQS的时候说过，acquire/release 和 acqureShared/releaseShared分别是独占式和共享式获取/释放资源的最顶层入口。这里为了能够理解清楚，再重新列出基类的方法。

### 2.2.1releaseShared（）

**这个方法是基类AQS的方法**

```java
public final boolean releaseShared(int arg) {
   	//调用自定义同步器的尝试释放资源方法
    if (tryReleaseShared(arg)) {
        //释放成功则尝试唤醒队列中等待的线程
        doReleaseShared();
        return true;
    }
    return false;
}
```
&ensp;&ensp;&ensp;&ensp;这个方法先调用自定义同步器的tryRleaseShared（int）方法尝试释放资源，因为是共享式的，所以释放成功之后（不需要等待state=0）需要通知队列中等待的资源尝试获取资源。注意这里前边传过来的入参是1，即只释放一个资源（实际上这个参数没有用到）。

#### 2.2.1.1tryReleaseShared（int）

```java
    protected boolean tryReleaseShared(int releases) {
        // Decrement count; signal when transition to zero
        //自旋
        for (;;) {
            //获取当前state值
            int c = getState();
            //如果当前state=0 则表示资源都已经释放了
            if (c == 0)
                return false;
            //将当前state值-1
            int nextc = c-1;
            //cas更新当前资源个数-1
            if (compareAndSetState(c, nextc))
                //更新成功则退出自旋返回true
                return nextc == 0;
        }
    }
```
&ensp;&ensp;&ensp;&ensp;这个方法主要完成的操作就是释放一个资源（文中的释放/获取对于state来说，获取表示state++，释放表示state--，可以反向理解为是对state来说），也就是使state的值-1，这个操作通过自旋CAS来保证一定会执行成功得到最后的结果。

#### 2.2.1.2doReleaseShared（）

**该方法为基类AQS的方法**

&ensp;&ensp;&ensp;&ensp;由前边对AQS的学习知道，当我们释放了一个资源的时候，我们要尝试去通知队列中第一个等待的线程去获取资源。对于共享式和独占式的区别是独占式是在释放之后资源个数=0时才去通知，共享式是释放了就去通知。这里是共享式释放资源的通知。这个方法也是基类的方法，就如同学习AQS时说的那样，自定义同步器不需要管队列的操作，只需要去重写对资源的获取和释放并处理好返回值即可。

```java
    private void doReleaseShared() {
        /*
         * Ensure that a release propagates, even if there are other
         * in-progress acquires/releases.  This proceeds in the usual
         * way of trying to unparkSuccessor of head if it needs
         * signal. But if it does not, status is set to PROPAGATE to
         * ensure that upon release, propagation continues.
         * Additionally, we must loop in case a new node is added
         * while we are doing this. Also, unlike other uses of
         * unparkSuccessor, we need to know if CAS to reset status
         * fails, if so rechecking.
         */
        //自旋
        for (;;) {
           	//获取队列头节点
            Node h = head;
            if (h != null && h != tail) {
                int ws = h.waitStatus;
                if (ws == Node.SIGNAL) {
                    if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                        continue;            // loop to recheck cases
                    unparkSuccessor(h);
                }
                else if (ws == 0 &&
                         !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                    continue;                // loop on failed CAS
            }
            if (h == head)                   // loop if head changed
                break;
        }
    }
```

这里相关内容不再继续展开了，主要的功能就是完成了对队列还活动着的线程进行了一个通知。

### 2.2.2小结

&ensp;&ensp;&ensp;&ensp;通过对源码的分析，我们可以看到，每个线程执行了countDown（）之后会对构造函数获取的state进行-1操作，执行完-1操作会通知队列中等待的线程尝试获取资源。这里我们也可以进行猜测，队列中等待的线程就是我们Demo中执行了await（）方法等待执行print任务的主线程。

## 2.3await（）

主线程为了等待子线程都执行完任务，调用了await（）方法

```java
public void await() throws InterruptedException {
    sync.acquireSharedInterruptibly(1);
}
```
可以看到这个方法是能够响应中断的。在响应了中断之后会抛出InterruptedException异常

### 2.3.1acquireSharedInterruptibly（）

这个方法是先尝试获取一个资源，如果获取失败了就寻找安全等待点自旋等待通知。等待过程中可以响应中断。

**该方法为基类AQS方法**

```java
public final void acquireSharedInterruptibly(int arg)
        throws InterruptedException {
    //被中断则抛出异常
    if (Thread.interrupted())
        throw new InterruptedException();
    //尝试获取资源
    if (tryAcquireShared(arg) < 0)
        //获取资源如果返回负数则寻找安全等待点
        doAcquireSharedInterruptibly(arg);
}
```
#### 2.3.1.1tryAcquireShared（int）

这个方法是自定义同步器重写的，也是在CountDownLatch中重写的我认为比较重要的一个方法。算是核心逻辑吧

```java
        protected int tryAcquireShared(int acquires) {
            return (getState() == 0) ? 1 : -1;
        }
```

&ensp;&ensp;&ensp;&ensp;哦豁，核心逻辑竟然只有一句话。就是判断当前共享资源值是不是等于0，如果等于0则返回1，否则返回-1。为什么说这里是核心逻辑呢，因为CountDownLatch实现的功能就是初始化对象的时候将共享资源个数设置成N，然后N个线程执行完之后-1，主线程等待时候需要判断共享资源个数，如果为0则表示其他线程都已经退下了。可以继续执行了，否则就需要进入等待队列等待。等待的过程中可以响应中断。

#### 2.3.1.2doAcquireSharedInterruptibly（）

```java
private void doAcquireSharedInterruptibly(int arg)
    throws InterruptedException {
    //将线程构造成Node节点加入到队列 标记为共享类型
    final Node node = addWaiter(Node.SHARED);
    //是否失败
    boolean failed = true;
    try {
        //自旋
        for (;;) {
            //获取前驱
            final Node p = node.predecessor();
            //如果前驱是头节点
            if (p == head) {
                //尝试获取资源（这个方法被重写了 实际并没有获取资源 而是检查资源个数是否为0 也就是上述的核心方法）
                int r = tryAcquireShared(arg);
                if (r >= 0) {
                    //如果资源个数为0了把当前节点设置成头
                    setHeadAndPropagate(node, r);
                    //插队导致前边的头结点无效 帮助其GC
                    p.next = null; // help GC
                    //没有失败
                    failed = false;
                    return;
                }
            }
            //循环遍历寻找一个安全等待点 即跳过队列排在前边无效的线程节点
            if (shouldParkAfterFailedAcquire(p, node) &&
                //自旋阻塞等待
                parkAndCheckInterrupt())
                //响应了中断抛出中断异常
                throw new InterruptedException();
        }
    } finally {
        if (failed)
            //如果失败了取消获取资源过程
            cancelAcquire(node);
    }
}
```
&ensp;&ensp;&ensp;&ensp;这个过程就是把自己加入到队列中并自旋等待的过程。等待过程中可以响应中断，这里会自旋判断自己是不是头节点了，与释放资源那里是遥相呼应的，因为资源释放之后会通知队列的头节点进行获取。内部的几个方法不再进行展开，相关AQS操作在AQS中已经阐述过。**这里最主要的还是上边的说的核心方法，方法名字是获取资源，但是实际重写操作内部完成的是判断资源个数是否为0**。

### 2.3.2小结

&ensp;&ensp;&ensp;&ensp;综上我们看到await（）方法的核心内容是查看当前资源个数是否为0，如果不为0则加入到等待队列中等待唤醒。等待过程可以响应中断。唤醒之后做了tryAcquireShared（）操作，依旧是判断资源是否为0，而不是字面意义去获取资源。（因为这里不是进行资源竞争，而是等待资源全部释放就可以继续完成后续的工作了。）

## 2.4await(long timeout, TimeUnit unit)

```java
    public boolean await(long timeout, TimeUnit unit)
        throws InterruptedException {
        return sync.tryAcquireSharedNanos(1, unit.toNanos(timeout));
    }
```

&ensp;&ensp;&ensp;&ensp;源码中还有个带timeout参数的await方法，顾名思义就是可以设置超时时间，比如主线程等待了1min之后子线程依然没有将state全部释放掉，那么就不等它了继续执行下边的方法。这里同样是结合了AQS的一些操作。具体就补展开阐述了。

# 三、总结

&ensp;&ensp;&ensp;&ensp;CountDownLatch在诸如主线程需要等待一系列子线程返回之后才继续执行的场景中使用，当然这种场景不仅仅局限于使用CountDownLatch。CountDownLatch使用方法是先用构造器构造一个对象，对象入参为要执行批量任务的线程个数，其实也可以是一个线程执行多个批量任务。所以这里如果理解为线程个数不是太准确，因为可能一个线程执行了多个任务，然后就需要等待多个任务的返回结果。所以这里合理的解释是构造函数的参数为要等待批量任务执行完成的批量个数。构造之后，每个任务执行之后调用一次countDown（）方法，等待执行最终任务的线程调用await（）方法。其底层原理是结合了AQS同步器完成，进行构造的时候将同步器的共享资源state设置成了N，然后每个任务执行完之后调用了countDown（）方法就将N-1，调用了await（）方法的线程被加入到了线程等待队列，自旋尝试获取资源，这里的获取资源是指自旋判断共享资源state是否等于0，即全部任务结束了。等待过程中可以响应中断。CountDownLatch还提供了一个可以设置超时时间的await（timeout）方法，以避免长时间阻塞。