---
title: 【Java编程思想】static关键字的四种用法
date: 2018-11-09 15:16:46
tags:
- Java 
- Think in Java
categories: 学习笔记
---

&ensp;&ensp;&ensp;&ensp;上一章说到了static关键字，static是Java中很重要的一个关键字，在一些场景下可以达到优化程序的效果。本文学习它的不同使用场景。在此之前先了解一下变量的类型。Java中变量分为两种，按作用域分为成员变量和局部变量。成员变量是在类中声明的，不属于任何方法，当前类中有效。局部变量是声明在方法中的，出了当前方法即超出作用域。接下来正文说一下static关键字的四中使用场景：

# 一、修饰成员变量

&ensp;&ensp;&ensp;&ensp;static最常用的作用就是修饰成员变量，被static修饰的成员变量也叫做类变量。它与普通成员变量的区别是，它在内存中只有一份拷贝，Java虚拟机只为其分配一次内存。通俗的讲就是，这个变量，不管被多少人使用，他们使用的都是同一个变量，彼此的修改会有影响。可以通过类名.变量的形式进行访问。而普通的成员变量则是没实例化一个对象就产生一个拷贝，虚拟机为每一个变量拷贝都分配了内存。所以类变量的用处一般在于进行变量共享的时候。

```java
public class TestStatic {
	private String name;
	private int age;
	public void print(){
		System.out.println("Name:" + name + "---Age:" + age);
	}
	public static void main(String[] args) {
		TestStatic ts1 = new TestStatic();
		TestStatic ts2 = new TestStatic();
		ts1.name = "zhangsan";
		ts1.age = 22;
		ts2.name = "lisi";
		ts2.age = 33;
		ts1.print();
		ts2.print();
	}
}
```

这是普通的成员变量的例子，ts1和ts2对于变量name和age分别拥有自己的副本，互相不影响，所以赋值之后，他们能得到他们期望的值。

![png1]([Java编程思想]static关键字的四种用法/png1.png)

接下来将成员变量age修改为类变量：

```java
public class TestStatic {
	private String name;
	private static int age;
	public void print(){
		System.out.println("Name:" + name + "---Age:" + age);
	}
	public static void main(String[] args) {
		TestStatic ts1 = new TestStatic();
		TestStatic ts2 = new TestStatic();
		ts1.name = "zhangsan";
		ts1.age = 22;
		ts2.name = "lisi";
		ts2.age = 33;
		ts1.print();
		ts2.print();
	}
}
```

这时由于age变成了类变量，因此ts1和ts2共用了一个变量副本，先赋值的会被后赋值的覆盖掉。

![png2]([Java编程思想]static关键字的四种用法/png2.png)

此外第二个示例代码中，静态成员变量使用了对象.变量的方式进行调用，这里编译器会给出警告，使用类名.方法之后警告就会解除。

![png3]([Java编程思想]static关键字的四种用法/png3.png)

# 二、修饰成员方法

&ensp;&ensp;&ensp;&ensp;static另一个作用是修饰类中的方法，其目的是可以通过类名.方法的形式调用方法，而避免频繁的创建对象。同时对于存储空间来说也只有一个，不同对象调用的是同一个方法：

```java
public class TestStatic {
	private String name;
	private static int age;
	public static void print(){
		System.out.println("Name:" + name + "---Age:" + age);
	}
	public static void main(String[] args) {
		TestStatic ts1 = new TestStatic();
		TestStatic ts2 = new TestStatic();
		ts1.name = "zhangsan";
		ts1.age = 22;
		ts2.name = "lisi";
		TestStatic.age = 33;
		TestStatic.print();
		TestStatic.print();
	}
}
```

将之前代码中的print方法改为使用static修饰之后，这个方法就可以不用对象.方法的形式调用了。

# 三、修饰代码块

&ensp;&ensp;&ensp;&ensp;我们先看一下对象的初始化过程：

```java
class Load {
	public Load(String msg){
		System.out.println(msg);
	}
}
public class TestStatic{
	Load ld1 = new Load("普通变量1");
	Load ld2 = new Load("普通变量2");
	static Load ld3 = new Load("静态变量3");
	static Load ld4 = new Load("静态变量4");
	public TestStatic(String msg){
		System.out.println(msg);
	}
	public static void main(String[] args) {
		TestStatic ts = new TestStatic("TestStatic 初始化");
	}
}
```

&ensp;&ensp;&ensp;&ensp;在类TestStatic中，我们初始化了两个普通成员变量和两个静态成员变量，并在main函数开始的时候初始化了TestStatic对象。结果如下：

![png4]([Java编程思想]static关键字的四种用法/png4.png)

&ensp;&ensp;&ensp;&ensp;静态成员变量最先被初始化，并且按照执行的先后顺序进行初始化。其次初始化的是成员变量，最后初始化的是构造方法。所以在创建一个对象的时候，最先被初始化的是静态成员变量。

&ensp;&ensp;&ensp;&ensp;在看另外一个调用了静态方法的例子：

```java
class Load {
	public Load(String msg){
		System.out.println(msg);
	}
}
public class TestStatic{
	Load ld1 = new Load("普通变量1");
	Load ld2 = new Load("普通变量2");
	static Load ld3 = new Load("静态变量3");
	static Load ld4 = new Load("静态变量4");
	public TestStatic(String msg){
		System.out.println(msg);
	}
	public static void staticFunc(){
		System.out.println("静态方法");
	}
	public static void main(String[] args) {
		TestStatic.staticFunc();
		System.out.println("@@@@@@@@@");
		TestStatic ts = new TestStatic("TestStatic 初始化");
	}
}
```

 我们在创建对象之前先调用了静态方法，结果如下:

![png5]([Java编程思想]static关键字的四种用法/png5.png)

&ensp;&ensp;&ensp;&ensp;我们可以看到，静态成员的初始化发生在创建对象之前，确切的说是在调用静态方法之前就已经被初始化了。并且，当我们创建对象的时候，原本被初始化过的静态成员变量跟静态方法没有再次被初始化。

&ensp;&ensp;&ensp;&ensp;这时我们的static的作用就是，修饰一段都需要被修饰为static的域。被static修饰的代码域，域中所有的内容都被当成static变量，且优先初始化。

```java
class Load {
	public Load(String msg){
		System.out.println(msg);
	}
}
public class TestStatic{
	Load ld1 = new Load("普通变量1");
	Load ld2 = new Load("普通变量2");
	static Load ld3;
	static Load ld4;
	static{
		ld3 = new Load("静态变量3");
		ld4 = new Load("静态变量4");
	}
	
	public TestStatic(String msg){
		System.out.println(msg);
	}
	public static void staticFunc(){
		System.out.println("静态方法");
	}
	public static void main(String[] args) {
		TestStatic.staticFunc();
		System.out.println("@@@@@@@@@");
		TestStatic ts = new TestStatic("TestStatic 初始化");
	}
}
```

修改之前的代码将静态成员变量放在由static修饰的域中，结果如下：

![png6]([Java编程思想]static关键字的四种用法/png6.png)

与分开修饰结果相同。

# 四、静态导入

&ensp;&ensp;&ensp;&ensp;前边三种都是比较常用的场景，还有一种不是太常用，是JDK1.5之后新加入的功能，导入一个带有静态方法的包，并将包修饰为static，从而在当前类中直接调用修饰的静态方法，就好像是自己的方法一样：

```
package com.chenxyt.java.test;
public class Printer {
	public static void print(String msg){
		System.out.println(msg);
	}
}
```

然后另一个包中使用import static导入这个类

```
package com.chenxyt.java.practice;
import static com.chenxyt.java.test.Printer.*;
public class TestStatic{
	public static void main(String[] args) {
		print("This is TestStatic");
	}
}
```

运行结果：

![png7]([Java编程思想]static关键字的四种用法/png7.png)

# 五、总结

&ensp;&ensp;&ensp;&ensp;static是Java语言中的一个很重要的关键字，主要用途有三个方面修饰成员变量，修饰成员方法以及修饰代码块。使用static修饰的成员变量在类加载的时候就已经被初始化了，它属于类变量，不属于某个对象，所有该类的实例化对象拥有同一个静态成员变量副本，常用的用途可以用它来做计数器。