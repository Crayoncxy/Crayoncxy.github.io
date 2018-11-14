---
title: 【Java编程思想】八：复用类
date: 2018-11-13 14:24:20
tags:
- Java 
- Think in Java
categories: 学习笔记
---

# 一、组合语法

&ensp;&ensp;&ensp;&ensp;代码复用是优化程序结构的一种必要手段，我们不需要重新编写代码，只需要获取一个类似的类在其基础上进行改动，保留一些原有的功能。改动的手段有组合跟继承两种。组合就是在一个新的类中引入原来的类的实例对象，这个对象是一个成员变量。如果不是一个对象，是一个基本数据类型，那么编译器会自动为其初始化。而当是一个对象的引用时，编译器则为其初始化为null，这样在使用这个对象时，就会发生错误。因此我们需要如下几个方法，初始化组合中的引用：

&ensp;&ensp;&ensp;&ensp;1.在定义对象的地方，也就是对象引用建立的时候直接指定对象，这意味着对象的初始化发生在构造器调用之前。

&ensp;&ensp;&ensp;&ensp;2.在类的构造器中，如第5章所述，我们可以通过构造器的形式，也就是new，初始化一个对象。

&ensp;&ensp;&ensp;&ensp;3.就在这样使用这些对象之前，也就是所谓的延时加载，当我们必须要使用这个对象的时候，发现没有初始化该对象呢，那么进行初始化，这种方式可以提供系统性能，降低资源消耗，在必须用到的时候才创建。

&ensp;&ensp;&ensp;&ensp;4.使用实例初始化，这种与构造器的相同之处都是在代码中显式的去创建一个对象，区别在于一个调用了构造器方法，一个赋值了一个实例。

下面分别举例说明四种初始化的方式：

&ensp;&ensp;&ensp;&ensp;1.在定义对象的地方，我们在类中定义了String的引用，直接初始化赋值：

```java
class Construct{
	public String cont = "Hello World";
}
```

&ensp;&ensp;&ensp;&ensp;2.在类的构造器中，如第五章的构造方法初始化：

```java
package com.chenxyt.java.practice;
class Construct{
	public String cont;
	Construct(String cont){
		this.cont = cont;
	}
}
public class ConstructTest {
	Construct construct = new Construct("Hello");
}
```

&ensp;&ensp;&ensp;&ensp;3.延迟加载，我们需要用到这个对象引用了，判断一下是否为null，然后再决定是否初始化：

```java
public class ConstructTest {
	public static String s;
	public static void main(String[] args) {
		if(s==null){
			s=new String("Hello");
		}
	}
}
```

&ensp;&ensp;&ensp;&ensp;4.使用实例进行初始化，如下边这种，我们没有使用构造方法而是直接赋值了个实例进行初始化：

```java
public class ConstructTest {
	public static String s;
	public static void main(String[] args) {
		s="Hello";
	}
}
```

# 二、继承语法

&ensp;&ensp;&ensp;&ensp;另一种复用类的手段就是继承，我们创建一个新的类，如果没有显示的指明继承哪个类，那么它默认继承自Object类，我们也可以通过extends关键字继承一个指定的类。继承表现的是一种is-a的关系或者说是like-a，被继承的类称作父类或者基类，继承之后新生成的类称为子类或者派生类。子类拥有父类中所有被public或protected域修饰的方法、属性。

```java
package com.chenxyt.java.practice;
class Father{
	public String Name;
	public int age;
	private double money;
	public void doPrint(){
		System.out.println("Hello World");
	}
}
public class Son extends Father {
	public static void main(String[] args) {
		Son son = new Son();
		son.Name = "Zhang San";
		son.doPrint();
		//money是父类私有的域，所以子类不能进行访问
		//son.money = 2014;
	}
}
```

# 三、代理

&ensp;&ensp;&ensp;&ensp;Java中还有一种基于组合与继承之间的关系，称作代理。但是实际上Java中并没有提供对它的直接支持。后续学习代理模式时会有专门的讲述。这里大概说一下，就是在使用组合的过程中，不提供成员对象的直接访问，而是通过一个新的方法暴露出来给其它使用者调用。避免直接暴露组合的对象。

```java
package com.chenxyt.java.practice;
class Construct{
	public void print(){
		System.out.println("print");
	}
	public void say(){
		System.out.println("say");
	}
	public void clean(){
		System.out.println("clean");
	}
}
public class Son{
	public Construct construct = new Construct();
	public void delegation(){
		construct.clean();
	}
	public static void main(String[] args) {
		Son son = new Son();
		son.delegation();
	}
}
```

# 四、结合使用组合和继承

&ensp;&ensp;&ensp;&ensp;同时使用组合和继承是很常用的事情，一般就是在一个子类中引用一个其它的类实例对象即可。诸如下边这种形式：

```java
package com.chenxyt.java.practice;
class Father{
	//---
}
class Mother{
	//---
}
public class Son extends Father{
	public Mother mother = new Mother();
	public static void main(String[] args) {
		//---
	}
}
```

&ensp;&ensp;&ensp;&ensp;关于继承还有一点说明，前文说到子类可以继承父类的全部，所以对于父类重载了的方法，子类也是可以继承并可以重载使用的。而且可以灵活的使用，避免了像C++中需要屏蔽父类该方法的方式。

```java
package com.chenxyt.java.practice;
class Father{
	public void doFunc(){
		//---
	}
	public void doFunc(String s){
		//--
	}
}
public class Son extends Father{
	public void doFunc(int i){
		//--
	}
	public static void main(String[] args) {
		Son son = new Son();
		son.doFunc();
		son.doFunc("H");
		son.doFunc(1);
	}
}
```

# 五、组合和继承之间选择

&ensp;&ensp;&ensp;&ensp;组合和继承都允许在新类中放置子对象，组合是显式的，而继承是隐式的。一般情况下，使用组合的地方多出是想使用其对象，就是在新类内部引用一个对象，然后利用这个对象完成一系列功能。而继承的使用场景主要是新类想使用原来类的一部分接口这种。在使用组合和继承的时候，要注意把我访问权限控制，因为一个完善的访问权限控制可以使程序更加健壮稳定。

# 六、protected关键字

&ensp;&ensp;&ensp;&ensp;前文说到访问权限控制，在一般情况下，为了保证不被他人修改自己的私有域，父类的只有自己可以有权使用的域跟方法会被设置为private，而在一些特殊情况下或许我们会放宽这种约束，即我的东西其他人都不可以用，但是我的子类可以用，这时候就是前文所说的protected关键字的作用了。被protected关键字修饰的域，可以被子类访问：

```java
package com.chenxyt.java.practice;
class Father{
	protected String s = "Hello";
}
public class Son extends Father{
	public static void main(String[] args) {
		Son son = new Son();
		System.out.println(son.s);
	}
}
```

运行结果，因为字符串s被声明为protected，所以在子类的对象引用son可以访问到：

![png1]([Java编程思想]八：复用类/png1.png)

# 七、向上转型

&ensp;&ensp;&ensp;&ensp;在继承的关系中，由于子类继承了父类所有的方法，所以发给父类方法的所有消息，子类都可以接收，注意这里是发给父类方法的所有消息，而不是子类调用了继承的方法。编译器会知道传递的参数引用属于哪一种类型，并将其子类对象引用的类型转换成父类，这个过程称作向上引用。

```java
package com.chenxyt.java.practice;
class Father{
	private static String s;
	Father(String s){
		this.s = s;
	}
	public static void doFunc(Father father){
		System.out.println(s);
	}
}
public class Son extends Father{
	Son(String s) {
		super(s);
		// TODO Auto-generated constructor stub
	}
 
	public static void main(String[] args) {
		Son son = new Son("Hello");
		Father.doFunc(son);
	}
}
```

&ensp;&ensp;&ensp;&ensp;如上所示，父类中有一个static的方法，并且方法的参数是父类的对象，在子类中直接通过类名.方法的形式调用了该方法，并传递了子类的对象引用作为参数。编译器没有报错，运行结果如下：

![png2]([Java编程思想]八：复用类/png2.png)

&ensp;&ensp;&ensp;&ensp;向上转型的概念来源于子类继承父类，子类在下，父类在上，由子类转换成父类，所以称作是向上转型。因为子类是父类的超集，所以向上转型总是安全的，相对也有向下转型，将在后文介绍。对于组合和继承，组合关系跟加方便且便于理解也常用，继承的关系虽然重要但不是说一定要多用。

# 八、Final关键字

&ensp;&ensp;&ensp;&ensp;final关键字与static同等重要，final顾名思义就是最终的，不可改变的。所以Java中final关键字也是指“这是无法改变的”。一般无法改变主要是出于两种理由，设计或者效率，这两种相差比较远，所以容易误用。这里主要从三个方面说明final关键字的用法。

&ensp;&ensp;&ensp;&ensp;**1.final数据：**当我们需要告诉编译器这一块数据区域不得改变的时候，比如一个固定的常量，或者一个在运行时候被初始化的值而我们却不希望它被改变，那么就可以使用final来修饰。比如类库中的某个成员变量，提供给开发者使用，但是不允许开发者修改这个变量在程序运行过程中的值，那么就可以使用final修饰。

```java
package com.chenxyt.java.practice;
public class FinalTest{
	private static final String ARG_NAME = "Hello World";
	public static void main(String[] args) {
		System.out.println(ARG_NAME);
	}
}
```

如果一个常量被修饰为static final 那么表明这个常量是不可被修改且只有一份的，并且它的命名规则要使用大写且用下划线分割所有单词。对于基本数据类型，final表示这个值不可以修改，对于对象的引用，表示这个引用不可以修改，也就是不可以指向别的对象。

&ensp;&ensp;&ensp;&ensp;**2.final参数**：Java中允许方法使用final修饰的参数，这意味着这个参数指向的对象不可以被修改

```
package com.chenxyt.java.practice;
class Gzino{
	public void doFunc(){
		//--
	}
}
public class FinalTest{
	public void with(final Gzino g){
		//error! 此处不可以修改该对象的引用
		//g = new Gzino();
		//可以使用该参数
		g.doFunc();
	}
	public void without(Gzino g){
		//没有被final修饰，可以使用
		g = new Gzino();
		g.doFunc();
	}
	public static void main(String[] args) {
		FinalTest ft = new FinalTest();
		ft.with(new Gzino());
		ft.without(new Gzino());
	}
}
```

&ensp;&ensp;&ensp;&ensp;**3.final方法：**使用final方法的原因有两个，第一个是把它锁定，以防任何继承它的类修改这个方法。第二个原因是提高效率，如果一个方法被声明为final方法，那么编译器会跳过这个方法的常规调用方式（参数入栈，跳到方法的执行部分，然后返回清理栈中参数，最后返回结果），并且以方法体中的实际代码副本代替方法调用，这样减小了方法调用的开销，但在方法过大的时候却不实用，因此在JDK1.5之后，取消了该种形式的作用，final方法只用来锁定防止其它类修改。

&ensp;&ensp;&ensp;&ensp;类中所有的private域都被隐式的注为private，因为不允许其它人修改。还有一种是final类，被final类修饰的方法我们认为这个类不能被其它人修改，也不能被继承，当你想达到这样的目的时，可以使用final类。

# 九、初始化及类加载

&ensp;&ensp;&ensp;&ensp;在许多语言中，程序在启动的过程中发生了加载，然后是初始化，最后是程序运行。Java不同，Java采用不同的加载机制， 每个类的编译文件都在它自己的代码文件中，该文件只有在用到该程序的时候才被加载。所以一般可以说“类的代码在初次使用时才发生加载“，这通常是指加载发生在创建第一个类对象时，但是当static域被访问时，也会发生加载。

```java
package com.chenxyt.java.practice;
class Insect{
	private int i = 9;
	protected int j;
	public Insect() {
		System.out.println("i=" + i + "--j=" + j);
		j=39;
	}
	private static int x1 = printInit("static Insect.x1 initialized");
	static int printInit(String s){
		System.out.println(s);
		return 53;
	}
}
public class LoadTest extends Insect{
	private int k = printInit("LoadTest.k initialized");
	public LoadTest(){
		System.out.println("k= " + k);
		System.out.println("j= " + j);
	}
	private static int x2 = printInit("static LoadTest.x2 initialized");
	public static void main(String[] args) {
		System.out.println("LoadTest constructor");
		LoadTest lt = new LoadTest();
	}
}
```

运行结果：

![png3]([Java编程思想]八：复用类/png3.png)

&ensp;&ensp;&ensp;&ensp;分析上面的程序，程序在启动的时候先查找main函数，在多数语言中，main函数都是程序的入口，发现main方法在LoadTest类中，这时开始加载LoadTest类，在加载的过程中，发现这个类有基类，于是他继续加载基类，如果还有基类那么继续加载另一个基类，因为子类中可能会用到基类的内容。然后基类中的static域初始化，这时第一行打印。然后子类中的static域初始化，这时第二行执行。这时必要的类资源已经加载完毕，可以进行对象创建，第三行打印。初始化对象，首先对象中的所有基本类型被赋初始值0，对象被赋值为null，这是通过将对象内存设置为二进制零实现的。然后基类的构造函数被调用，这里是自动调用的，也可以通过super方法直接调用，这时第四行被打印。基类构造器和子类构造器执行相同的方式经过相同的过程。基类构造器执行完毕之后，实例变量按顺序被创建，最后执行子类的构造方法，所以最后两行被打印。

# 十、总结

&ensp;&ensp;&ensp;&ensp;继承和组合都能从现有类中得到新的类型，区别在于组合一般是将现有类型作为底层实现的一部分使用，而继承复用的是接口。本章的重点内容是熟悉final关键字的用法以及类加载和初始化的过程，最后一点尤为重要。