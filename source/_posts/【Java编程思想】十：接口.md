---
title: 【Java编程思想】十：接口
date: 2018-11-14 14:35:13
tags:
- Java 
- Think in Java
categories: 学习笔记
---

# 一、抽象类和抽象方法

&ensp;&ensp;&ensp;&ensp;在前边关于多态的例子中，基类方法往往没有具体的实现，它存在的目的是为不同的子类提供统一的方法，通过动态绑定以及向上转型来完成子类需要的功能。为此，我们可以创建一个这样的类，只为子类提供接口，并且不允许这个类实例化对象，我们可以让这个类中的方法返回一个错误信息，但是这样存在一个问题，错误只能在运行时产生并且带来的影响不可预估。Java为我们提供了一个更加明确的方法，称为抽象方法，抽象方法顾名思义它是虚拟存在的，也就是它不能够被执行。这种方法是不完整的，它只有方法的声明，没有方法的实现，为了区分，使用abstract关键字表示一个方法是抽象方法。

```java
abstract void f();
```

&ensp;&ensp;&ensp;&ensp;包含抽象方法的类叫做抽象类，如果一个类包含一个或多个抽象方法，那么这个类叫做抽象类，同样使用abstract关键字修饰。抽象类是不安全的，因为它不完整，所以当试图用它实例化一个对象时，编译器会报错。如果一个类继承自一个抽象类，那么它必须为基类中所有的抽象方法提供一个具体的实现，否则这个子类也必须被定义为抽象类。我们也可以定义一个没有任何方法（包括没有抽象方法）的抽象类，这样做的目的是，这个类没有什么实际的意义，同时也不想让它能够创建对象。

&ensp;&ensp;&ensp;&ensp;下面一个示例看下抽象类和抽象方法的使用：

```java
package com.chenxyt.java.test;
abstract class Instrument{
	private int i;
	public String what(){
		return "Instrumet";
	}
	public abstract void play(String s);
	public void adjust(){};
}
class Wind extends Instrument{
	public String what(){
		return "Wind";
	}
	public void play(String s){
		System.out.println("Wind.play" + s);
	}
	public void adjust(){};
}
class Percussion extends Instrument{
	public String what(){
		return "Percussion";
	}
	public void play(String s){
		System.out.println("Percussion.play" + s);
	}
	public void adjust(){};
}
class Stringed extends Instrument{
	public String what(){
		return "Stringed";
	}
	public void play(String s){
		System.out.println("Stringed.play" + s);
	}
	public void adjust(){};
}
class Brass extends Instrument{
	public String what(){
		return "Brass";
	}
	public void play(String s){
		System.out.println("Brass.play" + s);
	}
	public void adjust(){};
}
class WoodWind extends Instrument{
	public String what(){
		return "WoodWind";
	}
	public void play(String s){
		System.out.println("WoodWind.play" + s);
	}
	public void adjust(){};
}
 
public class Music{
	static void tune(Instrument i ){
		i.play("finish");
	}
	static void tuneAll(Instrument[] e){
		for(Instrument i:e){
			tune(i);
		}
	}
	public static void main(String[] args) {
		Instrument[] iArray = {
				new Wind(),new Percussion(),new Brass(),new Stringed(),new WoodWind()
		};
		tuneAll(iArray);
	}
}
```

运行结果：

![png1]([Java编程思想]十：接口/png1.png)

&ensp;&ensp;&ensp;&ensp;如上可见，创建抽象类和抽象方法非常的有用，因为他们使类的抽象性更加明确，并告诉用户和编译器打算怎么样使用他们。抽象类还是一个很有用的重构工具，因为他们使得我们可以很容易的将公共方法沿着继承的层次向上移动。

# 二、接口

&ensp;&ensp;&ensp;&ensp;关键字使抽象的概念更加深入了一步。抽象类中可以允许抽象方法和普通方法共存，普通方法存在的目的是为所有继承的子类提供一个相同实现的方法。而接口创建了一个完全抽象的概念，接口内部不存在任何方法具体的实现。所有的实现都交由到实现这个接口的类完成。

&ensp;&ensp;&ensp;&ensp;接口使用interface关键字代替class关键字，访问权限控制与一个class相同，接口中可以包含域，但是这些域被隐式的定义为static和final型。

&ensp;&ensp;&ensp;&ensp;要想实现一个接口，就需要使用implements关键字显示的指明要实现哪个接口。接口中的方法必须被定义为public方法。实现接口的类要显示的编写实现接口中的所有方法，即便有些方法不需要实现，那也要如同接口一样写一个空的方法体。

&ensp;&ensp;&ensp;&ensp;修改上边抽象类的示例为接口实现：

```java
package com.chenxyt.java.test;
interface Instrument{
	public void play(String s);
}
class Wind implements Instrument{
	public String what(){
		return "Wind";
	}
	public void play(String s){
		System.out.println("Wind.play" + s);
	}
	public void adjust(){};
}
class Percussion implements Instrument{
	public String what(){
		return "Percussion";
	}
	public void play(String s){
		System.out.println("Percussion.play" + s);
	}
	public void adjust(){};
}
class Stringed implements Instrument{
	public String what(){
		return "Stringed";
	}
	public void play(String s){
		System.out.println("Stringed.play" + s);
	}
	public void adjust(){};
}
class Brass implements Instrument{
	public String what(){
		return "Brass";
	}
	public void play(String s){
		System.out.println("Brass.play" + s);
	}
	public void adjust(){};
}
class WoodWind implements Instrument{
	public String what(){
		return "WoodWind";
	}
	public void play(String s){
		System.out.println("WoodWind.play" + s);
	}
	public void adjust(){};
}
 
public class Music{
	static void tune(Instrument i ){
		i.play("finish");
	}
	static void tuneAll(Instrument[] e){
		for(Instrument i:e){
			tune(i);
		}
	}
	public static void main(String[] args) {
		Instrument[] iArray = {
				new Wind(),new Percussion(),new Brass(),new Stringed(),new WoodWind()
		};
		tuneAll(iArray);
	}
}
```

运行结果：

![png2]([Java编程思想]十：接口/png2.png)

# 三、完全解耦

&ensp;&ensp;&ensp;&ensp;只要一个方法操作的是类而非接口，那么你就只能使用这个类及其子类。如果你想将这个方法应用在不在此继承结构中的某个类，那么使用接口将很大程度的放宽这种限制。因此，它可以使我们编写可复用性更好的代码。
&ensp;&ensp;&ensp;&ensp;例如，有一个Processor类，它有一个name（）方法，还有一个process（）方法，该方法接受输入参数，修改输入的值然后进行输出。这个类作为基类被扩展，子类创建各种不同类型的Processor，在本例中，Processor子类通过process（）方法修改String对象的值，返回类型可以是协变类型，而非参数类型。

```java
package com.chenxyt.java.test;
import java.util.Arrays;
class Processor{
	public String name(){
		return getClass().getSimpleName();
	}
	Object process(Object input){
		return input;
	}
}
class UpCase extends Processor{
	String process(Object input){
		return input.toString().toUpperCase();
	}
}
class DownCase extends Processor{
	String process(Object input){
		return input.toString().toLowerCase();
	}
}
class Splitter extends Processor{
	String process(Object input){
		return Arrays.toString(input.toString().split(" "));
	}
}
public class Apply{
	public static void process(Processor p,Object s){
		System.out.println("Using Processor:" + p.name());
		System.out.println(p.process(s));
	}
	public static final String S = "Disagreement with beliefs is by definition incorrect";
	public static void main(String[] args) {
		process(new UpCase(), S);
		process(new DownCase(), S);
		process(new Splitter(), S);
	}
}
```

运行结果：

![png3]([Java编程思想]十：接口/png3.png)

&ensp;&ensp;&ensp;&ensp;前边学习多态的时候有过类似的例子，Apply.process()方法可以接收Processor类型跟它的子类，并将它应用到了Object对象，然后打印。像这种，根据继承关系，创建一个能够根据所传递的参数对象不同而具有不同行为的方法，称为策略设计模式。这类方法包含索要执行的方法中固定不变的部分（如本例的name（）方法），而“策略”包含变化的部分（如本例的process（））方法。策略就是传递的参数对象，它包含要执行的代码。这类Processor对象就是一个策略，在main（）方法中可以看到三种不同类型的策略应用到Obejct对象上。

&ensp;&ensp;&ensp;&ensp;下面有如下4个类，他们看起来也适用Apply.process()

```java
package com.chenxyt.java.practice;
public class Filter{
	public String name(){
		return getClass().getSimpleName();
	}
	public WaveForm process(WaveForm input){
		return input;
	}
}
```

&ensp;&ensp;&ensp;&ensp;Filter类，看上去与Processor类似，都有name（）方法和process（）方法，区别在于方法的参数类型和返回类型不同。

```java
package com.chenxyt.java.practice;
public class HighPass extends Filter{
	double cutoff;
	public HighPass(double cutoff){
		this.cutoff = cutoff;
	}
	public WaveForm prcess(WaveForm input){
		return input;
	}
}
```

&ensp;&ensp;&ensp;&ensp;HighPass类，继承自Filter类

```java
package com.chenxyt.java.practice;
public class BandPass extends Filter {
	double lowCutoff,highCutoff;
	public BandPass(double lowCutoff,double highCutoff){
		this.lowCutoff = lowCutoff;
		this.highCutoff = highCutoff;
	}
	public WaveForm process(WaveForm input){
		return input;
	}
}
```

&ensp;&ensp;&ensp;&ensp;BandPass类，继承自Filter类

```java
package com.chenxyt.java.practice;
public class WaveForm{
	private static long counter;
	private final long id = counter++;
	public String toString(){
		return "Wave Form" + id;
	}	
}
```

&ensp;&ensp;&ensp;&ensp;WaveForm类，前边方法的参数类型和返回类型。

&ensp;&ensp;&ensp;&ensp;Filter与Processor类具有相同的内部接口元素（两个类的方法名都相同），但是由于Filter类并非继承自Processor类，因此当Apply.process（）方法传入参数Filter的时候，由于Filter类的创建者并不知道要当做Processor类使用，并且它也不能通过向上转型的方式变成Processor类，因此不能将Filter类应用到Apply.process（）方法。这主要是因为Apply.process（）方法和Processor类的耦合度太高了，已经超出了所需要的程度。这就是Apply.process（）方法只能接收Processor类或者其子类，而面对新的类的时候，Apply.process（）方法就无能为力了，对其的复用也就被禁止了。

&ensp;&ensp;&ensp;&ensp;但是，正如前文所说，如果操作的是接口而不是类的时候，那么这些限制就会变得松动，使得你可以复用接口的Apply.process()方法，下面是修改为接口的版本。

```java
package com.chenxyt.java.practice;
public interface Processor{
	String name();
	Object process (Object input);
}
```

&ensp;&ensp;&ensp;&ensp;此时Processor类变成了一个接口，复用代码的形式就是之前继承它的类，可以改为实现它的接口，并且Filter类，也可以编程实现Processor类的接口，这样Apply.process（）方法的耦合度就降低了，并且支持了其它的类型。还有一种情况，假如一个类是被发现的，而不是被我们自己创建的，那么这个类就无法实现Processor接口，比如说，如果Filter类是在类库中的类，那么这个类就无法主动实现Processor接口，这时候可以使用适配器模式，在这个类的外部封装一层，作为适配器来实现要实现的接口。如下:

```java
package com.chenxyt.java.practice;
class FilterAdapter implements Processor{
	Filter filter;
	public FilterAdapter(Filter filter){
		this.filter = filter;
	}
	public String name(){
		return filter.name();
	}
	public WaveForm process(Object input){
		return filter.process((WaveForm)input)
	}
}
public class FilterProcessor{
	public static void main(String[] args) {
		WaveForm w = new WaveForm();
		Apply.process(new FilterAdapter(new LowPass(1.0)),w);
		Apply.process(new FilterAdapter(new HighPass(2.0)),w);
		Apply.process(new FilterAdapter(new BandPass(3.0,4.0)),w);
	}
}
```

&ensp;&ensp;&ensp;&ensp;在这种使用适配器的方式中，FilterAdapter的构造器接受了Filter参数，然后生成对应接口Processor的对象。
本节主要的内容是使用接口的方式将只有基类和其子类的使用方法解耦出来，便于程序更好的进行复用。

# 四、Java中的多重继承

&ensp;&ensp;&ensp;&ensp;C++中允许多重继承，并且每一个继承的类都可以有一个实现，Java中是不允许的，Java中可以实现多个接口，每个接口名字在implements后边用逗号隔开，并且，Java中只能继承一个类。下面的例子说明一个具体的类组合实现多个接口产生一个新类：

```java
package com.chenxyt.java.practice;
interface CanFight{
	void Fight();
}
interface CanSwim{
	void Swim();
}
interface CanFly{
	void Fly();
}
class ActionChracter{
	public void Fight(){
		//---
	};
}
class Hero extends ActionChracter implements CanFight,CanSwim,CanFly{
	public void Fly(){
		//---
	};
	public void Swim(){
		//--
	};
}
public class Adventure{
	public static void t(CanFight x){
		x.Fight();
	}
	public static void u(CanSwim x){
		x.Swim();
	}
	public static void v(CanFly x){
		x.Fly();
	}
	public static void w(ActionChracter x){
		x.Fight();
	}
	public static void main(String[] args) {
		Hero h = new Hero();
		t(h);
		u(h);
		v(h);
		w(h);	
	}
}
```

&ensp;&ensp;&ensp;&ensp;可以看到Hero类组合具体类ActionChracter和另外的三个接口，当通过这种方式将类和接口组合在一起时，这个类必须放在前边，接口放在后边，否则编译器会报错。同时我们注意到，CanFight接口与ActionChracter类中的Fight（）方法相同，而且Hero中并没有提供Fight（）的具体定义。可以扩展接口，当想要创建对象的时候，所有的定义必须都存在，即使Hero没有显示的定义Fight（）方法，由于其继承了ActionChracter类，所以定义随之而来，这使创建对象变成了可能。这里的意思是说，一个类实现了某些接口，这些接口中所有的定义在这个类中必须要有相关的实现（编译器会主动提示），然后因为这个类继承了一个类（ActionChracter），所以如果基类有实现了接口中的方法，那么子类就可以不显示的实现这个方法。（区别在于基类不是实现了这个方法，只是方法签名相同）

&ensp;&ensp;&ensp;&ensp;这个例子中，给出的四个方法分别使用接口作为了参数，所以在Hero作为参数传递的时候，它被依次进行了向上转型，Java中的接口设计，使得这项功能并不复杂。这个例子所展示的是使用接口的核心原因：为了能够向上转型为多个基本类型，提升程序的灵活性。使用接口的第二个原因与抽象类相同，防止程序员在使用的过程中创建该类的对象。当然关于这一点是使用抽象类还是接口，当要创建的类中没有任何方法定义和成员变量的定义是，选择接口是合适的，并且当知道某事物应当成为一个基类的时候，那么第一选择是应当使它成为接口。

# 五、通过继承扩展接口

&ensp;&ensp;&ensp;&ensp;接口也可以继承！没错，通过继承可以很容易的在接口中添加新的方法声明，还可以通过继承在新接口中组合数个接口。

```java
package com.chenxyt.java.practice;
interface Monster{
	void menace();
}
//新接口继承原来的接口
interface DangerousMonster extends Monster{
	void destroy();
}
interface Lethal{
	void kill();
}
//实现接口 要依次定义这个接口的方法以及它继承接口的方法  编译器自动补充
class DragonZill implements DangerousMonster{
	@Override
	public void menace() {
		// TODO Auto-generated method stub
	}
	@Override
	public void destroy() {
		// TODO Auto-generated method stub
	}
}
//接口可以多重继承
interface Vampire extends DangerousMonster ,Lethal{
	void drinkblood();
}
//继承多个接口 都要把定义实现
class VeryBadVampire implements Vampire{
 
	@Override
	public void destroy() {
		// TODO Auto-generated method stub
		
	}
 
	@Override
	public void menace() {
		// TODO Auto-generated method stub
		
	}
 
	@Override
	public void kill() {
		// TODO Auto-generated method stub
		
	}
 
	@Override
	public void drinkblood() {
		// TODO Auto-generated method stub
		
	}
}
```

代码中标注了，接口可以使用extends继承多个，但是这一形式不适用于普通的类。

&ensp;&ensp;&ensp;&ensp;这里说到了上边例子中的CanFight类和ActionChracter类都有一个相同的方法，如果方法只是名字相同，参数类型不同，返回类型不同，那么将带来逻辑上很大的问题。因此在继承、实现接口、覆盖或者重载的时候，应尽量避免重名的问题出现

# 六、适配接口

&ensp;&ensp;&ensp;&ensp;接口最吸引人的地方，就是允许同一个接口具有多个不同的实现。简单来说，就是一个接受接口类型的方法，而该接口的实现和向该接口传递的对象取决于方法的使用者。因此常用的方式就是前边的策略模式，此时你编写一个执行某些操作的方法，该方法接受一个同样是你指定的接口，你主要就是声明”你可以用任何你想要的对象来调用我的方法，只要你的对象遵循我的接口“这使你的方法更加灵活。

&ensp;&ensp;&ensp;&ensp;这里把我把上边适配接口的例子全部的代码撸了一遍，并分析了一下，具体的关于适配器模式后边会专门再学习一下，此处只是将书中的例子学习了一下

&ensp;&ensp;&ensp;&ensp;首先有个Processor接口，该接口有两个方法声明

```java
package com.chenxyt.java.practice;
public interface Processor{
	String name();
	Object process(Object input);
}
```

&ensp;&ensp;&ensp;&ensp;然后是个Apply类，这个类有个静态方法process，为了解耦，传递的参数为接口类型，接口的方法作用于一个Object对象

```java
package com.chenxyt.java.practice;
public class Apply {
	public static void process(Processor p,Object s){
		System.out.println("Using Processor" + p.name());
		System.out.println(p.process(s));
	}
}
```

这时我们发现了一个Filter类，因为这个类是发现的，所以看上去它跟Procesor接口有相同的方法，只是类型不同，所以可以直接实现该方法，但是由于这个类是已经写好了的。所以它不可以被修改了。

```java
package com.chenxyt.java.practice;
public class Filter{
	public String name(){
		return getClass().getSimpleName();
	}
	public Waveform process(Waveform input){
		return input;
	}
}
```

&ensp;&ensp;&ensp;&ensp;然后是一个Waveform类，这个类作为Filter类process方法的返回值，定义了个toString方法，所以在打印的时候会调用这个toString方法并把输入的内容返回

```java
package com.chenxyt.java.practice;
public class Waveform {
	private static long counter;
	private final long id = counter++;
	public String toString(){
		return "Waveform" + id;
	}
}
```

&ensp;&ensp;&ensp;&ensp;然后是Filter类的三个子类，分别实现了父类的process方法

```java
package com.chenxyt.java.practice;
public class LowPass extends Filter{
	double cutoff;
	public LowPass(double cutoff) {
		this.cutoff = cutoff;
	}
	public Waveform process(Waveform input){
		return input;
	}
}
```

```java
package com.chenxyt.java.practice;
public class HighPass extends Filter{
	double cutoff;
	public HighPass(double cutoff) {
		this.cutoff = cutoff;
	}
	public Waveform process(Waveform input){
		return input;
	}
}
```

```java
package com.chenxyt.java.practice;
public class BandPass extends Filter {
	double lowCutoff,highCutoff;
	public BandPass(double lowCutoff,double highCutoff){
		this.lowCutoff = lowCutoff;
		this.highCutoff = highCutoff;
	}
	public Waveform process(Waveform input){
		return input;
	}
}
```

&ensp;&ensp;&ensp;&ensp;这个时候问题来了，因为Appply类的process方法传参数是接口，这时Filter类已经存在，想直接使用Apply的process方法行不通，想实现Processor接口已经来不及。那么就要使用适配器模式啦。

```java
package com.chenxyt.java.practice;
 
class FilterAdapter implements Processor{
	Filter filter = new Filter();
	public FilterAdapter(Filter filter) {
		this.filter = filter;
	}
	@Override
	public String name() {
		// TODO Auto-generated method stub
		return filter.name();
	}
	public Waveform process(Object input) {
		// TODO Auto-generated method stub
		return filter.process((Waveform)input);
	}
}
public class FilterProcessor{
	public static void main(String[] args) {
		Waveform w = new Waveform();
		Apply.process(new FilterAdapter(new LowPass(1.0)),w);
		Apply.process(new FilterAdapter(new HighPass(2.0)),w);
		Apply.process(new FilterAdapter(new BandPass(3.0,4.0)),w);
	}
}
```

&ensp;&ensp;&ensp;&ensp;这里我简单的理解了一下这里的适配器模式，就是写了一个适配器的类，这个类实现了Processor接口，内部接受了Filter对象参数，然后生成了你想要的Processor接口对象，达到了预期的目的。这里有关具体的设计模式分析，后边会继续学习。

# 七、接口中的域

&ensp;&ensp;&ensp;&ensp;在接口中的域，会被自动的隐式转换为static final类型，所以接口就可以很便捷的创建一组常量值，也就是枚举。在JavaSE5之前，没有枚举的概念之前，可以使用接口来创建常量组。

```java
package com.chenxyt.java.practice;
public interface Months{
	int JANUARY = 1,FEBRUARY=2,MARCH=3,APRIL=4,
			MAY=5,JUNE=6,JULY=7,AUGUST=8,SEPTEMBER=9,OCTOBER=10,
			NOVEMBER=11,DECEMBER=12;
}
```

&ensp;&ensp;&ensp;&ensp;当然这种形式在后来已经被enum取代了。因为是final类型，所以必须显示的指定初始化的值，同时因为是static域，所以它们在第一次访问的时候被初始化，并且这些域不属于接口的一部分，它们的值存储在接口的静态存储区域。

# 八、嵌套接口

&ensp;&ensp;&ensp;&ensp;接口可以嵌套在类或者其它的接口中，个人觉得这种设计会使程序变得更加复杂不易读。

```java
package com.chenxyt.java.practice;
class A{
	interface B{
		void f();
	}
	public class BImp implements B{
		@Override
		public void f() {
			// TODO Auto-generated method stub
		}
	}
	public class BImp2 implements B{
		@Override
		public void f() {
			// TODO Auto-generated method stub
		}
	}
	public interface C{
		void f();
	}
	class CImp implements C{
		@Override
		public void f() {
			// TODO Auto-generated method stub
		}
	}
	private class CImp2 implements C{
		@Override
		public void f() {
			// TODO Auto-generated method stub
		}
	}
	private interface D{
		void f();
	}
	private class DImp implements D{
		@Override
		public void f() {
			// TODO Auto-generated method stub
		}
	}
	private class DImp2 implements D{
		@Override
		public void f() {
			// TODO Auto-generated method stub
		}
	}
	public D getD(){
		return new DImp();
	}
	private D dRef;
	public void reveiveD(D d){
		dRef = d;
		dRef.f();
	}
}
interface E{
	interface G{
		void f();
	}
	public interface H{
		void f();
	}
	void g();
	//强制必须为public
	//private interface I{};
}
 
public class NestingInterfaces{
	public class BImp implements A.B{
		@Override
		public void f() {
			// TODO Auto-generated method stub
		}
	}
	class CImp implements A.C{
		@Override
		public void f() {
			// TODO Auto-generated method stub
		}
	}
	//因为接口D是私有的 所以不能被实现
	//class DImp implements A.D{);
	class EImp implements E{
		@Override
		public void g() {
			// TODO Auto-generated method stub
		}
	}
	class EGImp implements E.G{
		@Override
		public void f() {
			// TODO Auto-generated method stub
		}
	}
	class EImp2 implements E{
		@Override
		public void g() {
			// TODO Auto-generated method stub
		}
		class EG implements E.G{
			@Override
			public void f() {
				// TODO Auto-generated method stub
			}
		}
	}
	public static void main(String[] args) {
		A a = new A();
		//D是private 不能实例化
		//A.D ad = new A.D();
		//getD()方法只能返回D
		//A.DImp2 di2 = a.getD();
		//private接口的域不能被访问
		//a.getD().f();
		//可以通过内部返回域的方法获取
		A a2 = new A();
		a2.reveiveD(a.getD());
	}
}
```

&ensp;&ensp;&ensp;&ensp;这里主要要说明的就是private interface接口的作用，就像A.D接口一样，它能够被实现为DImp的一个内部类，也同样可以像DImp2一样实现为public类，但是正如main方法中倒数第4行代码一样，A.DImp2只能被自己使用，因为你无法说你实现了一个private接口D，因此这个实现只是一种形式而已，它可以强制该方法的定义不带有任何类型信息，即不可以向上转型。所以我们在get方法中return new DImp2 的时候并没有获取到预期的值，因为DImp2是一个实现了private接口的public类，最终我们还是通过receiveD方法获取到了相应的实例。
&ensp;&ensp;&ensp;&ensp;接口E说明了接口之间的嵌套关系，因为接口内部所有元素都是public的，所以不能指明嵌套在内部的接口为private。NestingInterfaces展示了嵌套接口的几种形式，特别注意的是，在实现一个接口的时候，不需要实现其内部嵌套接口的方法。而且private接口不能在定义它的类外部被实现，比如上述代码中的A.D

# 九、接口与工厂

&ensp;&ensp;&ensp;&ensp;接口是实现多重继承的重要途径，而生成遵循某个接口对象的典型方式就是工厂方法设计模式。由此可见设计模式的重要性，我自己最近也在学习这一块的内容。希望能够有所提高。
&ensp;&ensp;&ensp;&ensp;使用工厂方法与直接调用构造器不同，我们在工厂对象上调用的是创建方法，而该工厂对象将生成接口的某个实现的对象。理论上来说，我们通过这种方式可以将我们的代码与接口的实现完全分离，这就使我们可以透明的将某个实现替换成另一个实现。如下示例：

```java
package com.chenxyt.java.practice;
interface Service{
	void method1();
	void method2();
}
interface ServiceFactory{
	Service getService();
}
class Implementation1 implements Service{
	Implementation1() {
		// TODO Auto-generated constructor stub
	}
	@Override
	public void method1() {
		// TODO Auto-generated method stub
		System.out.println("Implementation1 method1");
	}
	@Override
	public void method2() {
		// TODO Auto-generated method stub
		System.out.println("Implementation1 method2");	
	}
}
class Implementation1Factory implements ServiceFactory{
	@Override
	public Service getService() {
		// TODO Auto-generated method stub
		return new Implementation1();
	}
}
class Implementation2 implements Service{
	Implementation2() {
		// TODO Auto-generated constructor stub
	}
	@Override
	public void method1() {
		// TODO Auto-generated method stub
		System.out.println("Implementation2 method1");
	}
	@Override
	public void method2() {
		// TODO Auto-generated method stub
		System.out.println("Implementation2 method2");
	}
}
class Implementation2Factory implements ServiceFactory{
	@Override
	public Service getService() {
		// TODO Auto-generated method stub
		return new Implementation2();
	}
}
public class Factories {
	public static void serviceConsumer(ServiceFactory fact){
		Service s = fact.getService();
		s.method1();
		s.method2();
	}
	public static void main(String[] args) {
		serviceConsumer(new Implementation1Factory());
		serviceConsumer(new Implementation2Factory());
	}
}
```

&ensp;&ensp;&ensp;&ensp;这里如果不是使用工厂方法，代码中就要指定Service的确切类型，以便调用合适的构造器。使用工厂方法设计模式的原因是想要创建框架，提高代码的复用性。如下：

```java
package com.chenxyt.java.practice;
interface Game{
	boolean move();
}
interface GameFactory{
	Game getGame();
}
class Checkers implements Game{
	private int moves = 0;
	private static final int MOVES = 3;
	@Override
	public boolean move() {
		// TODO Auto-generated method stub
		System.out.println("Checkers moves" + moves);
		return ++moves != MOVES;
	}
}
class CheckersFactory implements GameFactory{
	@Override
	public Game getGame() {
		// TODO Auto-generated method stub
		return new Checkers();
	}
}
 
class Chess implements Game{
	private int moves = 0;
	private static final int MOVES = 4;
	@Override
	public boolean move() {
		// TODO Auto-generated method stub
		System.out.println("Chess move" + moves);
		return ++moves != MOVES;
	}
}
class ChessFactory implements GameFactory{
	@Override
	public Game getGame() {
		// TODO Auto-generated method stub
		return new Chess();
	}
}
public class Games {
	public static void PlayGame(GameFactory fact){
		Game s = fact.getGame();
		while(s.move()){
			//--
		}
	}
	public static void main(String[] args) {
		PlayGame(new CheckersFactory());
		PlayGame(new ChessFactory());
	}
}
```

&ensp;&ensp;&ensp;&ensp;如果Games类表示一段复杂的代码，那么这种方式就允许你在不同的游戏类型中复用这段代码。

# 十、总结

&ensp;&ensp;&ensp;&ensp;抽象类跟接口是将具体方法更加抽象的一种形式，这一章节主要讲了抽象类、抽象方法的形式以及使用场景，比较重要的一点是关于接口的使用，如何解耦，接口可以多重继承，接口可以嵌套等应用场景。关于这一章节中的设计模式，还要继续深入研究下去。