---
layout: posts
title: 【Java编程思想】十五：类型信息
date: 2018-11-20 09:39:48
tags:
- Java 
- Think in Java
categories: 学习笔记
---

# 一、为什么需要RTTI

&ensp;&ensp;&ensp;&ensp;RTTI（Run-Time-Type-Information），运行时类型信息。读《Java编程思想》这本书时是第一次知道这个词，于是先百度了一番，某乎上的讲解是这个概念最早是由本书的作者在《Think in C++》上提出的，实际上的意思与Java中的反射差不多。书中主要说的应用场景是在多态的过程中，基类的派生类使用基类提供的方法时，动态绑定以及向上转型相关，在最后阶段运行过程中找到自己对应的派生类。也正是有了RTTI，编译器才能正确的找到对应的派生类。上述思想是Java中多态思想的体现，多态也是Java中设计的关键思想，尽量使代码尽可能少的了解其具体的类型，而只与通用的类（基类）打交道。这种设计使代码逻辑更加简单、易于理解。

# 二、Class对象

&ensp;&ensp;&ensp;&ensp;要理解RTTI在Java中的工作原理，首先要知道类型信息在运行时是如何表示的。这项工作是由称为Class对象的一种特殊的对象表示的。面向对象思想告诉我们，万事万物都是对象，Java中除了基本的数据类型，其它的都是对象。那么如果我们有如下代码：

```java
Student s = new Student();
```

&ensp;&ensp;&ensp;&ensp;我们使用new Student（）创建了一个Student类的对象，那么Student类又是不是对象呢？没错，如前文所说，万事万物都是对象，这里Student就是Class的对象，也就是说任何声明定义的类，都是Class类的类对象。当编译器编译完一个类时，在同名的.class文件中就会保存这个类的Class对象。而为了生成这个对象，运行这个类的JVM会使用一个称作是“类加载器”的子系统。

&ensp;&ensp;&ensp;&ensp;类加载器子系统实际上可以包含一条类加载器链，但只有一个原生的类加载器，它是JVM实现的一部分，用来加载所谓的可信类，以及JavaAPI。如果还需要其它特殊的操作，比如支持Web服务应用，或者是在网络上下载类，那么就可以在类加载器链上挂载额外的类加载器。

&ensp;&ensp;&ensp;&ensp;所有的类在其第一次加载的时候，都被动态的加载到JVM中，当程序创建第一个类的静态成员引用时，就会被加载。这也说明类的构造器也是类的静态方法，即使在构造器之前并没有使用static关键字。因此使用new关键词创建的类的新的对象，也会被当做是类的静态成员引用。也正是如此，Java程序在运行之前并非完全加载，动态加载在诸如C++这类的静态加载语言是无法实现的。

一旦某个类的Class对象被载入内存，它就用来创建这个类的所有对象，如下示例说明这点：

```java
package com.chenxyt.java.practice;
class Candy{
	static{
		System.out.println("Loading Candy");
	}
}
class Gum{
	static{
		System.out.println("Loading Gum");
	}
}
class Cookie{
	static{
		System.out.println("Loading Cookie");
	}
}
public class SweetShop {
	public static void main(String[] args) {
		System.out.println("Inside main");
		new Candy();
		System.out.println("After Creating Candy");
		try{
			Class.forName("com.chenxyt.java.practice.Gum");
		}catch(ClassNotFoundException e){
			System.err.println("Gum Not found");
		}
		System.out.println("After Creating Gum");
		new Cookie();
		System.out.println("After Creating Cookie");
	}
}
```

运行结果：

![png1]([Java编程思想]十五：类型信息/png1.png)

&ensp;&ensp;&ensp;&ensp;可以看到Class对象仅在需要的时候被加载，static初始化是在类加载时候进行的。需要注意的是Class.forName（）这个方法，它是Class类的一个静态方法，所有的类都是Class类的对象，forName（）是取得Class对象引用的一种方法。它使用目标类名作为参数，返回Class对象的引用。如果找不到指定的类名则抛出ClassNotFound异常。无论何时你想获取到类的运行时使用的类型信息，就必须要获得恰当的Class对象的引用，Class.forName（）是实现此功能的捷径。因此你不需要为了获得Class对象的引用而持有该类型的对象。但是如果你已经拥有了一个指定类型的对象，那么就可以使用.getClass方法获取Class对象的引用。这个方法属于Object，它将返回表示该对象实际类型的Class对象引用。

Class还有很多实用的方法，如下:

```java
package com.chenxyt.java.practice;
interface HasBatteries{
	
}
interface WaterProof{
	
}
interface Shoots{
	
}
class Toy{
	Toy(){
		
	}
	Toy(int i){
		
	}
}
class FancyToy extends Toy implements HasBatteries,WaterProof,Shoots{
	FancyToy() {
		super(1);
	}
}
public class ToyTest {
	static void printInfo(Class cc){
		System.out.println("Class Name:" + cc.getName() + " Is Interface?" + cc.isInterface());
		System.out.println("Simple Name:" + cc.getSimpleName());
		System.out.println("Canonical Name:" + cc.getCanonicalName());
		System.out.println("----------");
	}
	public static void main(String[] args) {
		Class c = null;
		try{
			//获取指定类的类对象引用
			c = Class.forName("com.chenxyt.java.practice.FancyToy");
		}catch(ClassNotFoundException e){
			System.err.println("Class Not Found");
			System.exit(1);
		}
		printInfo(c);
		//获取该类实现的接口
		for(Class face:c.getInterfaces()){
			printInfo(face);
		}
		//获取该类的基类
		Class up = c.getSuperclass();
		Object obj = null;
		try{
			//Class类的构造器 用来构造不确定类型的对象
			obj = up.newInstance();
		}catch(InstantiationException e1){
			System.err.println(e1);
		}catch(IllegalAccessException e2){
			System.err.println(e2);
		}
		printInfo(obj.getClass());
	}
}
```

运行结果：

![png2]([Java编程思想]十五：类型信息/png2.png)

&ensp;&ensp;&ensp;&ensp;FancyToy继承了Toy并实现了三个接口，使用Class.forName（）方法创建一个FancyToy类的类对象，并指向引用c。getName（）产生全类名，getSimpleName（）获取不带包路径的类名，getCanonicalName（）获取全类名。isInterface（）判断当前类的类型是不是接口，Class.getInterfaces（）返回当前类的全部接口。如果你已经有了一个Class对象，还可以使用getSuperClass（）获取这个类的基类。Class的newInstance（）方法是实现了一个“虚拟的构造器”，意思是我不知道你的确切类型，但是你必须要正确的进行构造。使用newInstance（）创建的类，必须要带有默认构造器。

&ensp;&ensp;&ensp;&ensp;Java中除了以上两种Class.forName()和obj.getClass()可以产生类的对象引用，还有一种类字面常量的形式：

```java
FancyToy.class：
```

&ensp;&ensp;&ensp;&ensp;这样做不仅简单，而且更加安全，因为在编译时即可检查，无需使用try-catch语句，所以更加高效。类字面常量不仅可以应用与普通的类，还可以应用在基本数据类型，以及接口和数组中。此外对于基本数据类型的包装器类，它还有个TYPE字段，TYPE是一个引用，指向对应的基本数据类型的Class对象。

&ensp;&ensp;&ensp;&ensp;此处有一点非常重要需要注意，当使用.class来创建对象的引用时，不会自动的初始化该Class对象。为了使用类而做的准备工作实际上包含三个步骤：
**1.加载**：这是由类加载器执行的，该步骤将查找字节码（通常在classpath路径查找，但不是必须的），并从这些字节码中创建一个Class对象；
**2.链接**：在链接阶段验证类的字节码，为静态域分配存储空间，并且如果必要的话，会解析这个类创建的对其它类的所有引用；
**3.初始化**：如果该类具有超类，则对其进行初始化，执行静态初始化器和静态初始化块。
初始化被延迟到了对静态方法（构造器也是隐式的静态方法）或者非常数静态域进行首次引用时才执行：

```java
package com.chenxyt.java.practice;
import java.util.Random;
class Initable{
 static final int staticFinal = 45;
 static final int staticFinal2 = ClassInitialization.rand.nextInt(1000);
 static{
  System.out.println("Initializing Initable");
  System.out.println("Initable---------------");
 }
}
class Initable2{
 static int staticNonFinal = 145;
 static{
  System.out.println("Initializing Initable2");
  System.out.println("Initable2---------------");
 }
}
class Initable3{
 static int staticNonFinal = 54;
 static{
  System.out.println("Initializing Initable3");
  System.out.println("Initable3---------------");
 }
}
public class ClassInitialization {
 public static Random rand = new Random(45);
 public static void main(String[] args) throws ClassNotFoundException {
  Class initable = Initable.class;
  System.out.println("After Creating Initable Ref");
  System.out.println(Initable.staticFinal);
  System.out.println("------------------");
  System.out.println(Initable.staticFinal2);
  System.out.println(Initable2.staticNonFinal);
  Class initable3 = Class.forName("com.chenxyt.java.practice.Initable3");
  System.out.println("After Creating Initable3 Ref");
  System.out.println(Initable3.staticNonFinal);
 }
}
```

运行结果：

![png3]([Java编程思想]十五：类型信息/png3.png)

&ensp;&ensp;&ensp;&ensp;从结果中可以看出.class并不会引发初始化，相反使用Class.forName（）时则立刻完成了初始化的功能。同时，如果一个域被声明为static final的编译时常量，那么它可以不初始化类就被访问，就像Initable.staticFinal一样，但是仅仅被声明为static final并不会保证类不被初始化就能访问，就像Intable.staticFinal2那样，因为它不是编译时常量。
&ensp;&ensp;&ensp;&ensp;如果一个static域不是final的，那么在对它访问的时候，总是要求在读取之前先进行链接（为这个域分配存储空间）和初始化（初始化该存储空间），就像在对Initable2.staticNonFinal的访问中看到这样。
Class类允许使用泛型指定相应的类型，如下两种形式均可：

```java
package com.chenxyt.java.practice;
public class GenericClassReference {
	public static void main(String[] args) {
		Class intClass = int.class;
		Class<Integer> genericIntClass = int.class;
		genericIntClass = Integer.class;
		intClass = double.class;
		//类型不匹配
	//	genericIntClass = double.class;
		
	}
}
```

&ensp;&ensp;&ensp;&ensp;普通的类引用不会产生警告信息，并可以被重新指定为其它类型的引用。而泛型类的引用只能被指定为指定的类型。通过使用泛型语法，可以使编译器做一些额外的类型检查。如果我们希望放松这种限制，那么似乎可以使用：

```java
Class<Number> genericClass = int.class;
```

&ensp;&ensp;&ensp;&ensp;因为Integer类继承自Number类，但实际上并不可以正常工作，因为Integer Class对象不是Number Class对象的子类。为了放松这种类型的限制，我们使用了通配符，它是Java泛型的一部分。通配符就是“？”表示任何事物。

```java
Class<?> intClass = int.class:
```

&ensp;&ensp;&ensp;&ensp;尽管看起来这种形式与直接使用Class相同，但是它不会产生编译器警告信息。更确切的说使用通配符表示的是你并非是碰巧或者是由于疏忽而选择了一个非具体类的引用，而是你就是明确的选择了一个非具体的版本。使用通配符与extends关键字，可以有效的解决前边所说的继承的问题：

```java
Class<? extends Number> genericClass = int.class;
```

&ensp;&ensp;&ensp;&ensp;表示创建一个Number 或是它子类的类对象。

# 三、类型转换器先做检查

&ensp;&ensp;&ensp;&ensp;目前的类型转换信息有两种，一种是传统的类型转换，由RTTI确保类型转换，如果执行了一个错误的类型转换，那么将抛出一个ClassCastException异常，另一种是代表对象类类型的Class对象，通过查询Class对象的信息获取运行时的所有状态。RTTI在Java中还有第三种类型转换的形式，就是使用关键字instanceof，它返回布尔值，判断某个对象是不是某个特定类的实例。

```java
if(x instanceof Dog){
    (Dog)x.bark();
}
```

&ensp;&ensp;&ensp;&ensp;这种先进行判断然后再使用的方式显得很必然，比如在进行向下转型的时候，如果不先进行判断，很容易发生ClassCastException异常。此外也提供了动态判断的形式isInstanceof（）方法。

## 四、注册工厂

&ensp;&ensp;&ensp;&ensp;使用注册工厂的目的是将对象的创建工作交给类去完成，即创建一个工厂方法，然后进行多态的调用，从而为你创建恰当类型的对象。在如下的简单的版本中，工厂方法就是Factory接口中的Create方法。

```java
public interface Factory<T>{T create();}
```

&ensp;&ensp;&ensp;&ensp;所谓工厂方法，就是意味着我只提供一个创建实例的工厂方法，而无需每创建一个继承类就编写一个新的方法。关于工厂方法更多细节后边在设计模式模块专门再进行学习。

# 五、instanceof与Class的等价性

&ensp;&ensp;&ensp;&ensp;在查询类型信息的时候，instanceof与直接比较Class对象有一点很重要的区别，例子如下：

```java
package com.chenxyt.java.practice;
class BaseType{
	
}
class Derived extends BaseType{
	
}
public class FamilyVsExactType {
	static void test(Object x){
		System.out.println("Testing x of type" + x.getClass());
		System.out.println("x instance of BaseType" + (x instanceof BaseType));
		System.out.println("x instance of Derived" + (x instanceof Derived));
		System.out.println("BaseType.isInstance(x)" + BaseType.class.isInstance(x));
		System.out.println("Derived isInstance(x)" + Derived.class.isInstance(x));
		System.out.println("x.getClass == BaseType.class" + (x.getClass() == BaseType.class));
		System.out.println("x.getClass == Derived.class" + (x.getClass() == Derived.class));
		System.out.println("x.getClass.equals(BaseType.class)" + (x.getClass().equals(BaseType.class)));
		System.out.println("x.getClass.equals(Derived.class)" + (x.getClass().equals(Derived.class)));
	}
	public static void main(String[] args) {
		test(new BaseType());
		test(new Derived());
	}
}
```

运行结果：

![png4]([Java编程思想]十五：类型信息/png4.png)

&ensp;&ensp;&ensp;&ensp;可以看出，instanceof表示的是“你是这个类吗？或者你是这个类的子类吗？”而使用Class对象进行比较时，表示的则只有明确的类型信息，忽略了继承的关系。

## 六、反射：运行时类型信息

&ensp;&ensp;&ensp;&ensp;我理解的反射概念是程序在编译的时候并不知道具体的类型信息，直到程序运行时通过反射才获取到了准确的类信息。这里提供了几个方法支持获取准确的类信息，以便创建动态的代码。使用Class类的getMethods（）方法可以获取这个类所包含的所有方法，使用getConstructors（）方法可以获取这个类的所有构造函数。前文也提到过，使用Class.forName（）可以用来动态的加载类。它的生成结果在编译的时候是不可知的，因此所有的方法特征信息和签名都是在运行时被提取出来的。

# 七、动态代理

&ensp;&ensp;&ensp;&ensp;代理是常用的设计模式之一，它的作用是在基本的对象操作之外，增加一些其它额外的操作。比如我想使用一个对象，同事想了解这个对象的运行过程，那么这个运行过程的监控显然就不能放在基本的对象代码中。有点类似前面文章中提到的组合类的思想，新建一个代理类，代理类引用要使用的对象，然后增加一些新的功能。刚好前几天跟公司一个同事面了一个新人，同事问道代理模式之后自己做了一些阐述，可以把代理模式类比成是中介，中介的目的是卖东西的基础上赚钱，卖东西就是基本操作，赚钱就是中介也就是代理做的额外的操作。下面一个示例展示简单的代理模式：

先定义一个接口：

```java
package com.chenxyt.java.practice;
 
public interface Interface {
	void doSomeThing();
	void somethingElse(String arg);
}
```

然后是这个接口的实现类，也就是前边所说的真正要操作的运行的对象：

```java
package com.chenxyt.java.practice;
 
public class RealObject implements Interface {
 
	@Override
	public void doSomeThing() {
		// TODO Auto-generated method stub
		System.out.println("RealObject DoSomeThing");
	}
 
	@Override
	public void somethingElse(String arg) {
		// TODO Auto-generated method stub
		System.out.println("RealObject somethingElse");
	}
	
}
```

&ensp;&ensp;&ensp;&ensp;然后定义一个代理类，代理类实现了Interface接口，同时通过传参的形式传入了前边的实现类，在完成实现类功能的基础上，做了自己的操作：

```java
package com.chenxyt.java.practice;
 
public class SimpleProxy implements Interface{
	
	private Interface proxied;
	
	public SimpleProxy(Interface proxied) {
		// TODO Auto-generated constructor stub
		this.proxied = proxied;
	}
	@Override
	public void doSomeThing() {
		// TODO Auto-generated method stub
		System.out.println("Proxy DoSomething");
		proxied.doSomeThing();
		
	}
	@Override
	public void somethingElse(String arg) {
		// TODO Auto-generated method stub
		System.out.println("Proxy somethingElse");
		proxied.somethingElse(arg);
	}
	
}
```

&ensp;&ensp;&ensp;&ensp;最后是Main方法，因为consumer方法传参是Interface接口，所以任何实现了它接口的实体类都可以当做参数。这里演示了使用基本的实体类和使用代理的区别，代理在完成普通实体类的功能基础上打印了自己的操作内容：

```java
package com.chenxyt.java.practice;
 
public class SimpleProxyDemo {
	public static void consumer(Interface iface){
		iface.doSomeThing();
		iface.somethingElse("bobo");
	}
	public static void main(String[] args) {
		consumer(new RealObject());
		System.out.println("==============");
		consumer(new SimpleProxy(new RealObject()));
	}
}
```

![png5]([Java编程思想]十五：类型信息/png5.png)

&ensp;&ensp;&ensp;&ensp;在任何时刻你若想实现一些与“实际”对象分离的额外操作，或者你希望很容易能对这部分操作做出修改，那么使用代理模式无疑是最为方便的。

&ensp;&ensp;&ensp;&ensp;Java的动态代理比代理的思想更向前迈进了一步，因为它可以动态的创建代理，并且动态的处理对所代理方法的调用。动态代理所做的所有操作都会被重定向到单一的调用处理器上，它的工作是揭示调用的类型，并确定相应的对策。下面用动态代理重写上边的示例。

&ensp;&ensp;&ensp;&ensp;Java中要实现动态代理类，必须要继承InvocationHandle这个类，这个类内部嵌入的对象是要被实现的真正的对象，同样使用构造方法传入，这个类唯一的一个方法invoke，它有三个参数，第一个参数是生成的动态代理类。这里我个人理解，既然动态代理是动态的创建代理，那么这个参数固然是所创建的动态代理，第二个参数是传入的对象执行的方法，第三个参数是传入的参数。最后将请求通过Method.invoke（）方法，传入必要的参数，执行代理对象的方法。

```java
package com.chenxyt.java.practice;
 
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
 
public class DynamicProxyHandler implements InvocationHandler{
	
	private Object proxied;
	
	public DynamicProxyHandler(Object proxied) {
		// TODO Auto-generated constructor stub
		this.proxied = proxied;
	}
	@Override
	public Object invoke(Object proxy, Method method, Object[] args)
			throws Throwable {
		System.out.println("**** Proxy:" + proxy.getClass() + ",method:" + method + ",args " + args );
		if(args!=null){
			for(Object arg:args){
				System.out.println("  " + arg);
			}
		}
		// TODO Auto-generated method stub
		return method.invoke(proxied, args);
	}
}
```

客户端类：

```java
package com.chenxyt.java.practice;
 
import java.lang.reflect.Proxy;
 
public class SimpleDynamicProxyDemo {
	public static void consumer(Interface iface){
		iface.doSomeThing();
		iface.somethingElse("DIDI");
	}
	public static void main(String[] args) {
		RealObject real = new RealObject();
		consumer(real);
		System.out.println("=========");
		Interface proxy = (Interface)Proxy.newProxyInstance(Interface.class.getClassLoader(),new Class[]{Interface.class},new DynamicProxyHandler(real));
		consumer(proxy);
	}
}
```

&ensp;&ensp;&ensp;&ensp;客户端类使用Proxy.newProxyInstance（）方法创建了一个代理对象。第一个参数是一个加载器，这里用的是系统加载器，所以使用其它类的加载器结果也是一样的，第二个参数是代理对象要实现的接口，第三个参数是Handler对象，表示我这个代理对象在调用方法时，会映射关联到哪个Hnadler上去，这里是关联到了我们定义的DynamicProxyHandler上，然后转由它的invoke方法执行对象方法。

运行结果：

![png6]([Java编程思想]十五：类型信息/png6.png)

&ensp;&ensp;&ensp;&ensp;我们可以在代理方法invoke（）中对诸如方法名、参数进行过滤，只执行指定的方法。

&ensp;&ensp;&ensp;&ensp;最近项目中用到了AOP做日志记录的功能，Spring的AOP核心思想就是动态代理，现在更加理解了一些，简单表述一下就是在切入点处执行一写其它操作，如我这里是记录日志，然后执行正常的业务方法。记录日志就是脱离在实际业务之外的一些操作。

# 八、空对象

&ensp;&ensp;&ensp;&ensp;这里感觉不是很常用，大概理解了一下，就是当一个对象为null的时候，任何对这个对象的操作都会引发异常。为了避免这种情况，当一个对象为null的时候，我们定义一个空对象赋值给它。何谓空对象呢？就是一个不存在实际意义但是不会引发异常的对象。文中具体代码就不写了。

# 九、接口与类型信息

&ensp;&ensp;&ensp;&ensp;接口或者是面向接口编程的重要目的是实现隔离，也就是解耦。但是通过类型信息，这种耦合还是会传播出去。也就是接口对解耦来说，并不是绝对的。如下示例：

定义一个接口A，它有唯一一个方法f

```java
package com.chenxyt.java.practice;
 
public interface A{
	public void f();
}

```

接下来是接口的实现B，除了A中的方法，还有自己的方法g

```java
package com.chenxyt.java.practice;
 
public class B implements A{
 
	@Override
	public void f() {
		// TODO Auto-generated method stub
		System.out.println("method f");
	}
	public void g(){
	    System.out.println("method g");
	}
	
}
```

Main函数

```java
package com.chenxyt.java.practice;
 
public class InterfaceViolation {
	public static void main(String[] args) {
		A a = new B();
		a.f();
		System.out.println("a.getClass: " + a.getClass().getName());
		if(a instanceof B){
			B b = (B)a;
			b.g();
		}
	}
}
```

运行结果：

![png7]([Java编程思想]十五：类型信息/png7.png)

&ensp;&ensp;&ensp;&ensp;这里通过使用RTTI，发现a是被当做B类型实现的，因此通过将其转型成B，可以调用了B中新加的方法。

&ensp;&ensp;&ensp;&ensp;上面这种操作，完全合理并且可接受，但是提高了代码的耦合度，实际上并不允许这样做。解决办法是使用包访问权限来加以控制。

新定义一个实现类实现接口A：

```java
package com.chenxyt.java.practice;
 
class C implements A{
	@Override
	public void f() {
		// TODO Auto-generated method stub
	System.out.println("C.f");	
	}
	public void g(){
		System.out.println("C.g");
	}
	void u(){
		System.out.println("C.u");
	}
	protected void v(){
		System.out.println("C.v");
	}
	private void w(){
		System.out.println("C.w");
	}
}
	public class HidenC{
		public static A makeA(){
			return new C();
		}
	}
```

在另一个包中使用它：

```java
package com.chenxyt.java.test;
 
 
import java.lang.reflect.Method;
 
import com.chenxyt.java.practice.A;
import com.chenxyt.java.practice.HidenC;
 
public class HiddenImplementation {
	public static void main(String[] args) throws Exception {
		A a = HidenC.makeA();
		a.f();
		System.out.println("a.getClass: " + a.getClass().getName());
		//编译错误 受包访问权限控制
/*		if(a instanceof C){
			C c = (C)a;
			c.g();
		}*/
		callHiddenMethod(a,"g");
		callHiddenMethod(a,"v");
		callHiddenMethod(a,"u");
		callHiddenMethod(a,"w");
		
	}
	static void callHiddenMethod(Object a,String methodName) throws Exception{
		Method g = a.getClass().getDeclaredMethod(methodName);
		g.setAccessible(true);
		g.invoke(a);
	}
}
```

运行结果：

![png8]([Java编程思想]十五：类型信息/png8.png)

&ensp;&ensp;&ensp;&ensp;可以看到由于包访问权限的限制，在另一个包中获取不到了C新定义的方法。但是如果知道方法名，依然可以通过反射调用所有的方法。当然如果通过只发布编译之后的代码或许可以阻止这种情况发生，但是实际结果并不是这样。如下：

![png9]([Java编程思想]十五：类型信息/png9.png)

&ensp;&ensp;&ensp;&ensp;使用javap -private 可以看到包括private权限的方法在内的所有方法。因此任何人都可以获取到你这个类中方法名，然后通过反射调用他们。

接下来将接口实现为私有内部类看一下效果：

```java
package com.chenxyt.java.test;
import com.chenxyt.java.practice.A;
class InnerA{
	private static class C implements A{
		@Override
		public void f() {
			// TODO Auto-generated method stub
			System.out.println("C.f");
		}
		public void g(){
			System.out.println("C.g");
		}
		void u(){
			System.out.println("C.u");
		}
		protected void v(){
			System.out.println("C.v");
		}
		private void w(){
			System.out.println("C.w");
		}
	}
	public static A makeA(){
		return new C();
	}
}
public class InnerImplemention {
	public static void main(String[] args) throws Exception {
		A a = InnerA.makeA();
		a.f();
		System.out.println("a.getClass " + a.getClass().getName());
		HiddenImplementation.callHiddenMethod(a,"g");
		HiddenImplementation.callHiddenMethod(a,"u");
		HiddenImplementation.callHiddenMethod(a,"v");
		HiddenImplementation.callHiddenMethod(a,"w");
	}
}
```

运行结果：

![png10]([Java编程思想]十五：类型信息/png10.png)

&ensp;&ensp;&ensp;&ensp;可见使用内部类还是不能躲避反射的查找。

接下来试一下匿名类：

```java
package com.chenxyt.java.test;
 
import com.chenxyt.java.practice.A;
 
class AnoymousA{
	public static A makeA(){
		return new A(){
			@Override
			public void f() {
				// TODO Auto-generated method stub
				System.out.println("C.f");
			}
			public void g(){
				System.out.println("C.g");
			}
			void u(){
				System.out.println("C.u");
			}
			protected void v(){
				System.out.println("C.v");
			}
			private void w(){
				System.out.println("C.w");
			}
		};
	}
}
public class AnoymousImplemention {
	public static void main(String[] args) throws Exception {
		A a = AnoymousA.makeA();
		a.f();
		System.out.println("a.getClass" + a.getClass().getName());
		HiddenImplementation.callHiddenMethod(a,"g");
		HiddenImplementation.callHiddenMethod(a,"u");
		HiddenImplementation.callHiddenMethod(a,"v");
		HiddenImplementation.callHiddenMethod(a,"w");
	}
}
```

运行结果：

![png11]([Java编程思想]十五：类型信息/png11.png)

&ensp;&ensp;&ensp;&ensp;可见匿名类也被反射给找到了。即便是private域的方法也逃不过反射的到来。

&ensp;&ensp;&ensp;&ensp;书中最后一段讲述，如果有人通过这种所谓的投机的方式获取内部私有的方法，那么他就应该接受这种方法改变之后带来的后果。

# 十、总结

&ensp;&ensp;&ensp;&ensp;反射这一章节，看的前后拖的时间比较长了，基本理解了反射的概念，比如在运行时获取类型信息，通过class对象可以获得这个类的全部信息啊一些，因为文中涉及到一些设计模式，所以又补充了一下设计模式的知识。