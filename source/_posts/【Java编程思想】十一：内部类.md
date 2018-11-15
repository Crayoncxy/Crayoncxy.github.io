---
title: 【Java编程思想】十一：内部类
date: 2018-11-15 10:23:00
tags:
- Java 
- Think in Java
categories: 学习笔记
---

# 一、创建内部类

&ensp;&ensp;&ensp;&ensp;将一个类定义在另一个类的内部，这就是内部类。内部类与组合是不同的概念。

&ensp;&ensp;&ensp;&ensp;创建一个内部类：

```java
package com.chenxyt.java.test;
public class Parcell {
	class Contents{
		private int i = 11;
		public int value(){
			return i;
		}
	}
	class Destination{
		private String label;
		public Destination(String whereto) {
			// TODO Auto-generated constructor stub
			label = whereto;
		}
		String readLabel(){
			return label;
		}
	}
	public Contents contents(){
		return new Contents();
	}
	public Destination destination(String s){
		return new Destination(s);
	}
	public void ship(String dest){
		Contents c = new Contents();
		Destination d = new Destination(dest);
		System.out.println(d.readLabel());
	}
	public static void main(String[] args) {
		Parcell p1 = new Parcell();
		p1.ship("Inner Class");
		Parcell p2 = new Parcell();
		Contents c = p2.contents();
		Parcell.Destination d = p2.destination("Class Inner");
	}
}
```

&ensp;&ensp;&ensp;&ensp;如上我们创建了一个内部类，内部类与其它类的区别在于将类隐藏在了另一个类的内部，同时如contents方法所示，外部类的方法还可以返回一个指向内部类的引用，这也是很常见的一种用法。此外我们看到main（）方法中创建的内部类对象是使用外部类的引用关联创建的，这一点在下一节中会说到。

![png1]([Java编程思想]十一：内部类/png1.png)

# 二、链接到外部类

&ensp;&ensp;&ensp;&ensp;上边的代码似乎只展示了内部类与其它类名字和组织结构的区别，内部类还有其它的用途。当我们创建了一个内部类对象，此对象就与制造它的外围对象之间有了一种关联，所以它能访问其外围对象的所有成员，而不需要任何特殊条件。此外，内部类还拥有其外围类的所有元素的访问权。

```java
package com.chenxyt.java.practice;
interface Selector{
	boolean end();
	Object current();
	void next();
}
public class Sequence {
	private Object[] items;
	private int next = 0;
	public Sequence(int size){
		items = new Object[size];
	}
	public void add(Object x){
		if(next<items.length){
			items[next++] = x;
		}
	}
	private class SequenceSelector implements Selector{
		private int i = 0;
		@Override
		public boolean end() {
			// TODO Auto-generated method stub
			return i == items.length;
		}
		@Override
		public Object current() {
			// TODO Auto-generated method stub
			return items[i];
		}
		@Override
		public void next() {
			// TODO Auto-generated method stub
			if(i<items.length){
				i++;
			}
		}
	}
	public Selector selector(){
		return new SequenceSelector();
	}
	public static void main(String[] args) {
		//初始化一个长度为10的数组
		Sequence sequence = new Sequence(10);
		//数组赋值 每个域赋值为下标编号
		for(int i = 0;i<10;i++){
			sequence.add(Integer.toString(i));
		}
		//创建接口对象 返回接口实现的为内部类的对象
		Selector selector = sequence.selector();
		//判断当前数组下标是否已到最大
		while(!selector.end()){
			//返回当前数组域的值
			System.out.print(selector.current() + " ");
			//数组下标+1
			selector.next();
			}
	}
}

```

&ensp;&ensp;&ensp;&ensp;上边的代码很好的展示了这一点，外部类Sequence创建了个数组，内部类SequenceSelector实现了Selector接口获取数组内容并操作数组，可以看到这个类的实现方法访问了外部类的private域。 

![png2]([Java编程思想]十一：内部类/png2.png)

&ensp;&ensp;&ensp;&ensp;所以内部类自动拥有对其外围类所有成员的访问权限，那么这是如何做到的呢？当某个外围类的对象创建一个内部类的对象的时候，这个内部类对象必然会秘密捕获一个外围类对象的引用，也就是这个引用来选择外围类的成员。这里所有的细节都交给了编译器来处理。内部类的对象只能在其与外部类的对象相关联的时候才能被创建（在static方法中），构建内部类对象时，需要一个指向其外围类对象的引用，如果编译器访问不到这个引用就会报错，不过绝大多数情况下这种都不需要我们操心。这里也就解释了上一节关于内部类初始化方式不同的原因。意思就是在static方法中，不能通过直接的new 构造函数的形式创建内部类，因为这种形式没有将内部类与外部类对象做关联，要先创建一个外部类的对象，然后使用该对象的引用创建内部类对象。当然如果内部类是静态的，那么就没有这种要求了。

![png3]([Java编程思想]十一：内部类/png3.png)

![png4]([Java编程思想]十一：内部类/png4.png)

&ensp;&ensp;&ensp;&ensp;我们将上一个示例中的ship方法改成static修饰，原来的使用构造器创建内部类对象的方法就报错了。因为它没有找到关联的外部类对象引用。然后我们把这个内部类Contents改成static修饰，则编译器就不报错了。或者使用main函数中的形式，先创建一个外部类的对象，然后使用这个对象的引用去创建内部类对象。

# 三、使用.this和.new

&ensp;&ensp;&ensp;&ensp;如果我们需要在内部类生成外部类对象的引用，那么可以使用外部类名.this的形式，这里如果只使用this，则返回的是内部类对象的引用。

```java
package com.chenxyt.java.practice;
public class DoThis {
	void f(){
		System.out.println("This is outClass's method");
	}
	public class Inner{
		public DoThis outer(){
			return DoThis.this;
			//return this;
		}
	}
	public Inner inner(){
		return new Inner();
	}
	public static void main(String[] args) {
		DoThis dt = new DoThis();
		Inner in = dt.inner();
		in.outer().f();
	}
}
```

运行结果：

![png5]([Java编程思想]十一：内部类/png5.png)

&ensp;&ensp;&ensp;&ensp;如上在outer（）方法中使用return this 会提示Dothis类型无法转换成Inner类型。
&ensp;&ensp;&ensp;&ensp;如前文说到，我们不能使用new直接创建内部类对象，我们需要使用外部类对象的引用创建，这里可以使用外部类对象的引用.new语法进行创建。

```java
package com.chenxyt.java.practice;
public class DoThis {
	public class Inner{
		public DoThis outer(){
			return DoThis.this;
			//return this;
		}
	}
	public static void main(String[] args) {
		DoThis dt = new DoThis();
		Inner in = dt.new Inner();
	}
}
```

&ensp;&ensp;&ensp;&ensp;只有内部类是静态内部类或者在非static方法中，我们才可以通过new的形式直接进行创建。

# 四、内部类与向上转型

&ensp;&ensp;&ensp;&ensp;当将内部类向上转型为基类时，尤其是转型为接口时，内部类就有了用武之地。这是因为我们可以使内部类也就是接口的实现完全不可见也不可用，得到的只是基类或者接口的引用，从而更好的隐藏了实现的细节。

&ensp;&ensp;&ensp;&ensp;我们先创建两个接口：

```java
package com.chenxyt.java.practice;
public interface Destination{
	String readLabel();
}
```

```java
package com.chenxyt.java.practice;
public interface Contents {
	int value();
}
```

&ensp;&ensp;&ensp;&ensp;然后创建一个类，并在这个类的内部创建两个内部类去实现上边的接口:

```java
package com.chenxyt.java.practice;
class Parcell4{
	private class PContents implements Contents{
		private int i = 11;
		@Override
		public int value() {
			// TODO Auto-generated method stub
			return i;
		}
	}
	protected class PDestination implements Destination{
		private String label;
		private PDestination(String whereto) {
			// TODO Auto-generated constructor stub
			label = whereto;
		}
		@Override
		public String readLabel() {
			// TODO Auto-generated method stub
			return label;
		}
	}
	public Destination destination(String s){
			return new PDestination(s);
		}
	public Contents contents(){
			return new PContents();
		}
}
public class TestParcell {
	public static void main(String[] args) {
		Parcell4 p = new Parcell4();
		Contents c = p.contents();
		Destination d = p.destination("Inner Class");
		System.out.println(d.readLabel());
		System.out.println(c.value());
		//因为PContents是private 所以不能被访问
		//Parcell4.PContents pc = p.new PContents();
	}
}
```

&ensp;&ensp;&ensp;&ensp;Parcell4中增加了一些新的东西，首先内部类PContents是private，除了Parcell4没有人能访问它，所以main函数最后一行编译不能通过。其次PDestination是protected的，所以除了该类本身和其子类还有同一个包中的类，其它类不能访问。因此客户端如果想访问这些实现，就受到了限制。不过我们可以看到，main函数的第二、第三行都实现了转型，也就是虽然不可见，但是不影响使用接口的实现。因此private的内部类提供了一种设计思路，通过这种方式完全阻止了依赖任何类型的编码，并且完全隐藏了实现的细节，并且由于不能访问任何新增加的、原本不属于公共接口的方法，因此接口的扩展就是没有价值的了。

![png6]([Java编程思想]十一：内部类/png6.png)

# 五、在方法和作用域内的内部类

&ensp;&ensp;&ensp;&ensp;有些时候我们可以将内部类创建在方法的作用域里或者是其它任何地方的作用域中，这么做有两个理由：

&ensp;&ensp;&ensp;&ensp;1.如前所示，实现了某个类型的接口，可以创建并返回接口的引用。

&ensp;&ensp;&ensp;&ensp;2.要解决一个复杂的问题，需要一个类来辅助解决，但是又不希望这个类是公开的。

&ensp;&ensp;&ensp;&ensp;在方法的作用域内部创建的内部类称为局部内部类：

```java
package com.chenxyt.java.practice;
public class Parcel5 {
 public Destination destination(String s){
  class PDestination implements Destination{
   private String label;
   private PDestination(String whereTo){
    label=whereTo;
   }
   @Override
   public String readLabel() {
    // TODO Auto-generated method stub
    return label;
   }
  }
  return new PDestination(s);
 }
 public static void main(String[] args) {
  Parcel5 p5 = new Parcel5();
  Destination d = p5.destination("Area Inner Class");
  System.out.println(d.readLabel());
 }
}
```

&ensp;&ensp;&ensp;&ensp;如上所示，destination方法中的内部类实现了Destination接口，PDestination类是destination方法内部的类，所以其它地方不能访问。return语句向上转型返回了Destination，它是PDestination类的接口，也就是基类。

![png7]([Java编程思想]十一：内部类/png7.png)

&ensp;&ensp;&ensp;&ensp;下面的例子展示在任意作用域中使用内部类：

```java
package com.chenxyt.java.practice;
public class Parcel6 {
	private void internalTracking(boolean b){
		if(b){
			class TrackingSkip{
				private String id;
				TrackingSkip(String s){
					id = s;
				}
				String getSkip(){
					return id;
				}
			}
			TrackingSkip ts = new TrackingSkip("SLIP");
			String s = ts.getSkip();
			System.out.println(s);
		}
		//因为内部类在if（b）的作用域内 此处已经超过了作用范围 所以不可以使用
		//TrackingSkip ts1 = new TrackingSkip("SLIP1");
	}
	public void track(){
		internalTracking(true);
	}
	public static void main(String[] args) {
		Parcel6 p = new Parcel6();
		p.track();
	}
}
```

&ensp;&ensp;&ensp;&ensp;上边的例子主要是想在方法中判断如果入参为true则实现一个类的功能，而这个类又不想被外部可见，所以在if中创建了内部类。

![png8]([Java编程思想]十一：内部类/png8.png)

# 六、匿名内部类

&ensp;&ensp;&ensp;&ensp;现在有如下个例子：

```java
package com.chenxyt.java.practice;
public class Parcel7 {
 public Contents contents(){
  return new Contents(){
   private int i = 11;
   @Override
   public int value() {
    // TODO Auto-generated method stub
    return i;
   }
  };
 }
 public static void main(String[] args) {
  Parcel7 p7 = new Parcel7();
  Contents c = p7.contents();
 }
}
```

&ensp;&ensp;&ensp;&ensp;在此处，contents方法内部要返回一个Contents对象的时候，我们突然加了一个类的定义，这个类没有名字，它实现了Contents接口，也就是我们实际上创建了一个继承自Contents类的匿名类对象，于是这个return对象的引用就变成了一个来自向上转型的Contents引用。上述这个匿名内部类是下面这种形式的一种简化。

```java
package com.chenxyt.java.practice;
public class Parcel7 {
	public class MyContents implements Contents{
		private int i = 11;
		@Override
		public int value() {
			// TODO Auto-generated method stub
			return i;
		}
	}
	public Contents contents(){
		return new MyContents();
	}
	public static void main(String[] args) {
		Parcel7 p7 = new Parcel7();
		Contents c = p7.contents();
	}
}
```

&ensp;&ensp;&ensp;&ensp;上述代码使用了默认的无参构造器，下边的示例展示当基类的构造器为有参数的构造器时，匿名内部类应当如何创建：

&ensp;&ensp;&ensp;&ensp;基类：

```java
package com.chenxyt.java.practice;
public class Wrapping {
	private int i;
	public Wrapping(int x){
		i=x;
	}
	public int value(){
		return i;
	}
}
```

&ensp;&ensp;&ensp;&ensp;匿名内部类：

```java
package com.chenxyt.java.practice;
public class Parcel8 {
	public Wrapping wrapping(int x){
		return new Wrapping(x){
			public int value(){
				return super.value() * 11;
			}
		};
	}
}
```

&ensp;&ensp;&ensp;&ensp;只需要传递合适的参数到基类的构造器中即可，虽然Wrapping只是一个普通的实现类，但是他还是被其导出类当做了公共接口来使用。

&ensp;&ensp;&ensp;&ensp;如第一个例子所示，匿名内部类中的域可以进行初始化操作。但是，当匿名内部类的域要使用外部对象的引用时，需要强行将函数参数的引用设置为final才可以。

```java
package com.chenxyt.java.practice;
public class Parcel8 {
	public Wrapping wrapping(final int x){
		return new Wrapping(x){
			public int value(){
				return super.value() * x;
			}
		};
	}
}
```

# 七、嵌套类

&ensp;&ensp;&ensp;&ensp;前边我们说的内部类，都是必须要有外部类关联的，也就是这些内部类有个隐式的引用，指向外部类。如果我们不需要这种关联，那么就可以将内部类显示的声明为static的，这种内部类称为嵌套类。嵌套类意味着：

&ensp;&ensp;&ensp;&ensp;1.要创建嵌套类的对象，并不依赖外部对象

&ensp;&ensp;&ensp;&ensp;2.不能从嵌套类的对象中访问非静态的外围类对象

&ensp;&ensp;&ensp;&ensp;嵌套类与普通的内部类还有一个区别，普通内部类的字段与方法，只能放在类的外部层次上，所以普通的内部类不能有static数据和static字段，也不能包含嵌套类，而嵌套类可以包含所有这些东西。

```java
package com.chenxyt.java.practice;
 
public class Parcel11 {
	private static class ParcelContents implements Contents{
		private int i = 11;
		@Override
		public int value() {
			// TODO Auto-generated method stub
			return i;
		}
	}
	protected static class ParcelDestination implements Destination{
		private String label;
		private ParcelDestination(String whereTo){
			label = whereTo;
		}
		@Override
		public String readLabel() {
			// TODO Auto-generated method stub
			return label;
		}
		public static void f(){
			//
		}
		static int x = 8;
		static class AnotherLevel{
			public static void funx(){
				//
			}
			static int y = 11;
		}
	}
	public static Destination destination(String s){
		return new ParcelDestination(s);
	}
	public static Contents contents(){
		return new ParcelContents();
	}
	public static void main(String[] args) {
		Contents c = new ParcelContents();
		Destination d = new ParcelDestination("dss");
		Destination d1 = destination("d1");
	}
}
```

&ensp;&ensp;&ensp;&ensp;如上，在main方法中，没有任何Parcel11对象是必须的，我们可以自由的创建内部类对象。
&ensp;&ensp;&ensp;&ensp;在正常情况下，不能在接口内部放置任何代码，但是嵌套类可以作为接口的一部分，放在接口中的任何域都是public static的，所以放入的类是嵌套类，甚至我们可以使用此类实现外部接口。如果你想要创建某些公共的代码，并让它可以被某个接口的所有不同实现所共有，那么这种情况就最好不过了。

```java
package com.chenxyt.java.practice;
public interface ClassInnerInterface {
	void howd();
	class Test implements ClassInnerInterface{
		@Override
		public void howd() {
			// TODO Auto-generated method stub
			System.out.println("其它不同实现的公共方法");
		}
		public static void main(String[] args) {
			new Test().howd();
		}
	}
}
```

&ensp;&ensp;&ensp;&ensp;如上我们在内部类中加入了一个main函数，这个main可以用来测试每一个我们编写的类，编译过程中会生成一个独立的class文件，名称如下。如果在生产项目上，我们可以简单的在编译好的文件中删除该class文件即可。

![png9]([Java编程思想]十一：内部类/png9.png)

&ensp;&ensp;&ensp;&ensp;一个内部类被嵌套多少层并不重要，重要的是它能够透明的访问它所嵌入的外围类的所有成员，即便是被定义为private的域。

```java
package com.chenxyt.java.practice;
class ManyInner {
	private void f(){
		//---
	}
	class A{
		private void g(){
			//---
		}
		public class B{
			void h(){
				f();
				g();
			}
		}
	}
}
public class MNA{
	public static void main(String[] args) {
		ManyInner mi = new ManyInner();
		ManyInner.A a = mi.new A();
		ManyInner.A.B b = a.new B();
		b.h();
	}
}
```

&ensp;&ensp;&ensp;&ensp;如上可见，在嵌套了多层的内部类B中调用方法f（）和方法g（）并不需要任何其它附加条件，即便他们被定义为private

# 八、为什么需要内部类

&ensp;&ensp;&ensp;&ensp;一般来说，内部类继承自某个类或实现某个接口，内部类的代码操作创建它的外围类的对象，所以可以认为内部类提供了一种进入它外围类的窗口。如果我们只是需要一个对接口的引用，那么为何不使用外围类去实现那个接口呢？答案是如果这样能满足需求，那就需要这样做。内部类实现接口与外部类实现接口的区别在于后者不是总能享用到接口带来的方便，有时需要用到接口的实现。所以使用内部类最吸引人的原因：

**&ensp;&ensp;&ensp;&ensp;每个内部类都能够独立的继承自一个（接口的）实现，所以无论外围类是否已经继承了某个（接口的实现），对于内部类都没有影响。**

&ensp;&ensp;&ensp;&ensp;内部类有效的实现了“多重继承”，如果在一个类中要使用两个接口，那么使用单一类和内部类看起来没有什么区别（因为单一类可以直接实现多个接口，此处不写例子了），而如果这两个接口换成是抽象类或者是具体的类，那么由于Java不支持多重继承的原因，这里使用单一类显然不能解决问题了，而内部类恰好可以有效的解决这个问题，看似是个“多重继承”。

```java
package com.chenxyt.java.practice;
//要继承的类1
class F{
	//---
}
//要继承的类2
abstract class G{
	//---
}
//外部类继承A 内部匿名类继承B
class H extends F{
	G makeG(){
		return new G(){
			//--
		};
	}
}
public class MutiExtends {
	static void takeF(F f){
		//--
	}
	static void takeG(G g){
		//--
	}
	public static void main(String[] args) {
		H h = new H();
		takeF(h);
		takeG(h.makeG());
	}
}
```

&ensp;&ensp;&ensp;&ensp;如果不是要解决类似上边的“多重继承”问题，那么可以不实用内部类，但是使用内部类还可以获得一些其它的特性。

&ensp;&ensp;&ensp;&ensp;1.内部类可以有多个实例，每个实例都有自己的状态信息，并且与其外部类的对象信息相互独立。
&ensp;&ensp;&ensp;&ensp;2.在单个外围类中，可以让多个内部类继承或实现多个基类。
&ensp;&ensp;&ensp;&ensp;3.创建内部类的时刻并不依赖于外围对象的创建。（个人不太理解这里，因为前边说了非嵌套类在static方法中使用时，需要关联一个外部对象的引用，不知道这里具体指的是什么。）
&ensp;&ensp;&ensp;&ensp;4.内部类并没有令人迷惑的is-a关系，它是一个独立的实体。

# 九、内部类的继承

&ensp;&ensp;&ensp;&ensp;内部类由于与外部类之间有一个隐式的引用关联关系，所以在继承内部类的时候，要显示的说明他们之前的关联。

```java
package com.chenxyt.java.practice;
class withInner{
	class Inner{		
	}
}
public class InheritInner extends withInner.Inner{
	InheritInner(withInner wi){
		wi.super();
	}
	public static void main(String[] args) {
		withInner wi = new withInner();
		InheritInner ii = new InheritInner(wi);
	}
}
```

&ensp;&ensp;&ensp;&ensp;如上，在子类的构造函数中要传入继承内部类的外部类的引用。

# 十、内部类可以被覆盖吗

&ensp;&ensp;&ensp;&ensp;如果一个类继承了另一个外部类，那么基类中的内部类会发生覆盖吗？

```java
package com.chenxyt.java.practice;
class Egg{
	private Yolk y;
	protected class Yolk{
		public Yolk(){
			System.out.println("Egg.Yolk");
		}
	}
	public Egg(){
		System.out.println("New Egg");
		y = new Yolk();
	}
}
public class BigEgg extends Egg{
	public class Yolk{
		public Yolk(){
			System.out.println("BigEgg.Yolk");
		}
	}
	public static void main(String[] args) {
		new BigEgg();
	}
}
```

&ensp;&ensp;&ensp;&ensp;基类中使用了默认的构造函数，并创建了一个内部类对象，子类中“覆盖”了内部类，所以预期的结果应该是使用子类中覆盖之后的内部类构造器。

![png10]([Java编程思想]十一：内部类/png10.png)

&ensp;&ensp;&ensp;&ensp;但是实际结果显然不是这样的，它还是走了正常的逻辑流程，说明子类并没有覆盖基类中内部类，这两个内部类彼此独立，在自己的命名空间中。当我们想进行类似“覆盖”内部类的功能时，可以明确的继承内部类，然后覆盖其方法。

# 十一、局部内部类

&ensp;&ensp;&ensp;&ensp;定义在方法体中内部类称为局部内部类：

```java
package com.chenxyt.java.practice;
interface Counter{
	int next();
}
public class LocalInnerClass {
	private int count = 0;
	//局部内部类
	Counter getCounter(final String name){
		class LocalCounter implements Counter{
			public LocalCounter(){
				System.out.println("LocalCounter Constructor");
			}
			@Override
			public int next() {
				// TODO Auto-generated method stub
				System.out.println(name);
				return count++;
			}
		}
		return new LocalCounter();
	}
	//匿名类
	Counter getCounter2(final String name){
		return new Counter(){
			{
				System.out.println("Counter Constructor");
			}
			@Override
			public int next() {
				// TODO Auto-generated method stub
				System.out.println(name);
				return count++;
			}
		};
	}
	public static void main(String[] args) {
		LocalInnerClass lc = new LocalInnerClass();
		Counter c1 = lc.getCounter("Local Counter");
		Counter c2 = lc.getCounter2("Annoy Counter");
		for(int i=0;i<5;i++){
			System.out.println(c1.next());
		}
		for(int i=0;i<5;i++){
			System.out.println(c2.next());
		}
	}
}
```

&ensp;&ensp;&ensp;&ensp;我们分别使用局部内部类和匿名类实现了相同的计数功能。

![png11]([Java编程思想]十一：内部类/png11.png)

&ensp;&ensp;&ensp;&ensp;它们具有相同的行为能力，那么既然局部类在方法体外部是看不见的，那么为什么不使用匿名类呢？唯一的理由是我们需要一个已命名的构造器，或者重载构造器，而匿名类只能用于实例初始化。也就是说因为没有名字，所以没有可见的构造器。所以使用局部内部类的另一个原因就是需要不止一个该内部类的对象。

# 十二、内部类标识符

&ensp;&ensp;&ensp;&ensp;由于每个类都会产生一个.class文件，其中包含如何创建该类型的全部信息。（此信息产生一个“meta-class”,叫做class对象）所以内部类也一定会有个.class文件，它们有规范的命名规则，外围类加上“$”加上内部类的名字。

&ensp;&ensp;&ensp;&ensp;如果是匿名内部类，编译器会简单的产生一个数字作为标识符，如果内部类是嵌套在别的内部类里，那么就继续使用"$"符号。

# 十三、总结

&ensp;&ensp;&ensp;&ensp;1.内部类是定义在一个类内部的类，这个类可以在方法中，也可以在方法外。内部类可以访问到其外部类的所有域。
&ensp;&ensp;&ensp;&ensp;2.内部类与外部类之间的关联关系是使用一个隐式的外部类引用，所以在创建内部类时，需要先创建一个外部类引用进行关联。这种形式出现在当内部类不是static修饰并且在static方法域中创建内部类对象引用的时候。
&ensp;&ensp;&ensp;&ensp;3.在内部类中，要使用外部类.this才可以返回外部类的对象引用，如果使用this只是返回了内部类的对象引用。当我们创建了一个外部类对象引用时，可以使用引用.new 内部类（）的形式创建内部类对象。
&ensp;&ensp;&ensp;&ensp;4.内部类可以用来向上转型实现接口，这种方式有效的建立了接口与实现的隔离，可以使实现完全不可见，不可修改。
&ensp;&ensp;&ensp;&ensp;5.在方法和作用域里的类，有两个作用，一个是如前所示用来实现接口并返回。另一个原因是我想创建一个类辅助我解决问题，但是不想这个类可见。
&ensp;&ensp;&ensp;&ensp;6.使用return new xx（）{} 在{}内部定义类的一些域可以创建一个实现或继承xx的匿名类，这个匿名类没有名字，也就没有构造函数。匿名类使用的外部方法引用需要被修饰为final。
&ensp;&ensp;&ensp;&ensp;7.如前所示，内部类的创建需要与外部类进行关联。如果我们不需要进行关联，那么可以将内部类修饰为static，这种称为嵌套类。嵌套类与外部类彼此独立。
&ensp;&ensp;&ensp;&ensp;8.内部类可以实现类似“多重继承”。
&ensp;&ensp;&ensp;&ensp;9.内部类与外部类的引用有关联，所以在继承内部类的时候需要显示的在构造函数中引用外部类的引用，以说明这种关联。
&ensp;&ensp;&ensp;&ensp;10.外部类被继承之后，内部类没有发生特别的变化，也就是它不会被覆盖，如果在子类中重新定义同名的内部类，这会被认为是第二个类，与之前的内部类彼此在不同的命名空间，没有关联。
&ensp;&ensp;&ensp;&ensp;11.局部内部类是定义在方法中的，作用与匿名类相同，但是有构造函数，可以进行构造函数重载。
&ensp;&ensp;&ensp;&ensp;12.所有的类都有标识符，内部类的标识符为外部类名字$内部类名字。