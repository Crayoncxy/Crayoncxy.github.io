---
title: 【Java编程思想】七：访问权限控制
date: 2018-11-13 14:06:48
tags:
- Java 
- Think in Java
categories: 学习笔记
---

 &ensp;&ensp;&ensp;&ensp;一个优秀的程序员是通过不断的重构代码让自己的程序变得更加易用、可读和完善的。在重构修改的过程中，如果是一个类库编写人员，那么怎么样保证自己修改的部分不会影响到客户端编写人员（即使用这个类库的程序员）呢？同时也要避免他们对自己类库内部的程序进行改动。Java中提供了访问权限控制的概念，提供了三种不同级别的访问控制，访问开放程度由高到低依次为“public”、“protected”“private”，这样就能区分哪些内容是可用的，哪些内容是不可用的，从而将变动的事物与不变的事物区分开来。那么如何将所有的构建捆绑到一个内聚的类库单元中呢？Java提供了package加以控制，而访问权限控制的作用会因为类库是否在一个相同的package还是不同的package受到影响。

# 一、包：库单元

&ensp;&ensp;&ensp;&ensp;包内包含一组类，它们在单一的名字空间下被组织在了一起。声明一个类所属的包使用package关键字，同时在另一个包中的类要访问其它包中的类使用import关键字导入要使用的包。这种方式可以在一定程度上避免重名的问题，因为包的名字要避免重名，而不同包内的类是可以根据具体的需求命相同的名字。包有效的将不同类的内容进行了隔离，同时也可以相互联系。
如下是com.chenxyt.java.test包中的Printer类，类中定义了一个print方法：

```java
package com.chenxyt.java.test;
public class Printer {
	public void print(String msg){
		System.out.println(msg);
	}
}
```

在另一个包com.chenxyt.java.practice中的PackageTest类，类中使用print方法：

```java
package com.chenxyt.java.practice;
import com.chenxyt.java.test.Printer;
public class PackageTest{
	public static void main(String[] args) {
		Printer printer = new Printer();
		printer.print("This is Package Test");
	}	
}
```

结果如下：

![png1]([Java编程思想]七：访问权限控制/png1.png)

看一下程序的目录结构:

![png2]([Java编程思想]七：访问权限控制/png2.png)

# 二、Java访问权限修饰词

&ensp;&ensp;&ensp;&ensp;public：所有可见，被public修饰的内容在同一个包中的所有类都可见。同时Java提供默认的访问权限，即不被任何修饰符修饰的内容默认为public权限。

&ensp;&ensp;&ensp;&ensp;private：私有可见，只有该类可见，该类的对象都不可见。如果一个类的构造函数被声明为private，那么就不能通过这个类的构造函数来进行初始化对象。

&ensp;&ensp;&ensp;&ensp;protected:受保护的可见，与private不同，除了只有自己的类可见之外，该类的继承者也可见被修饰的域。除此之外还可以被当前包的类访问，但是其它包的类不可以访问，即便是使用了import的关键字

访问权限控制对程序结构控制的重要手段。

# 三、接口和实现

&ensp;&ensp;&ensp;&ensp;这里没有详细的介绍Java中的接口跟实现，主要是基于访问权限控制来说的。一般的类开发者，为了方便他人使用，会在具体方法实现外部建立一层接口，只提供接口给外部开发人员调用，而不提供具体实现的方法。

# 四、类的访问权限

&ensp;&ensp;&ensp;&ensp;在Java中，访问权限控制也可以确定包中的哪些类可以被访问，也就是说可以用来修饰类，一个文件中最多只能有一个使用public修饰的类。如果希望客户端程序员使用该类，并可以创建对象，那么就可以将该类修饰为public。并且被修饰为public的类必须要与该文件的名字完全相同

# 五、总结

&ensp;&ensp;&ensp;&ensp;本章主要学习的是Java中的三种访问权限，熟练的掌握public、private和protected三种类型的概念以及应用场景将能更好的提高程序的健壮性和稳定性。