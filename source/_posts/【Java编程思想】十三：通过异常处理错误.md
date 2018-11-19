---
title: 【Java编程思想】十三：通过异常处理错误
date: 2018-11-16 09:24:08
tags:
- Java 
- Think in Java
categories: 学习笔记
---

&ensp;&ensp;&ensp;&ensp;错误的理想时机是在程序的编写过程中，但是一些业务逻辑错误，在编写过程中很难发现。这些问题就只能在程序运行的过程中解决。这就需要程序在运行过程中发生错误的时候能够准确的提供信息给某个接收者，以便可以正确的处理错误。

&ensp;&ensp;&ensp;&ensp;Java中提供了一种非常优秀的处理错误的手段，被称为“异常处理”，下文讲述如何编写正确的异常处理程序，并将展示方法出现问题时，如何自定义异常。

# 一、概念

&ensp;&ensp;&ensp;&ensp;C语言以及其它早期的编程语言都是采用一种约定俗称的模式，比如在发生错误的时候，返回一个错误标记，或者设置一个特殊的变量来记录错误。这种形式需要增加大量的判断以及额外的代码无疑使程序更加繁重不利于构建。

&ensp;&ensp;&ensp;&ensp;解决的方法是，用强制规定的形式来消除错误处理过程中随心所欲的因素。“异常”这个词有“我对此感到意外”的意思，当错误问题出现了，你可能不知道出现在哪里，或者出现了什么错误，也不知道怎么解决。那么就停下来，将这个问题提供给更高的环境，看看有没有正确的解决方案。使用异常的另一个好处是，它能够明显的降低代码的复杂程度，避免了大量的错误检查，只需要在一个特定的地方进行异常捕获，并且不需要做任何判断，异常捕获区能够捕获所有发生的错误。这种异常的处理方式与之前的错误处理方式相比，完全的将“正常做的事儿”与“出现问题怎么办”隔离开来。是代码的读写变得更加井井有条。

# 二、基本异常

&ensp;&ensp;&ensp;&ensp;“异常情形”是指阻止当前方法或者作用域继续执行的问题，要把异常情形与普通问题区分开来，普通问题是指在当前的错误收集情况下，能够解决这个问题并继续执行正常的程序。而对于异常情形，这个程序就不能够正常的执行下去了。
&ensp;&ensp;&ensp;&ensp;除法是一个简单的例子，除数有可能为0，所以先进行检查很有必要。但除数为0如果是一个意外的值，你也不清楚该如何处理，那么抛出异常就显得尤为重要了，而不是顺着原来的路走下去。
&ensp;&ensp;&ensp;&ensp;当抛出异常之后有几件事儿会随之发生，同 Java中其它对象的创建相同，将使用new在堆上创建对象，然后当前程序的执行路径被终止了，因为发生了异常，不能继续执行下去，并且从当前执行的环境中弹出异常对象的引用。此时，异常处理机制接管程序，并开始寻找一个恰当的地方继续执行程序，这个恰当的地方就是异常处理程序。它的任务是使程序从错误的状态中恢复，要么换一种方式运行，要么继续运行下去。
&ensp;&ensp;&ensp;&ensp;举一个抛出异常的简单例子，对于一个对象引用t，传递给你的时候可能没有被初始化，所以在使用这个对象引用调用执行方法之前，进行合理的判断是非常有必要的。可以创建一个代表错误信息的对象，并且将它从当前环境中抛出，这样就把错误异常抛到更大的环境中去了。所以一个异常，看起来是这样的：

```java
if(t==null){
    throw new NullPointerException();
}
```

&ensp;&ensp;&ensp;&ensp;上面代码中，我们判断当前对象引用是否没有进行初始化，如果没有进行初始化，那么就创建一个NullPointerException（）对象，然后使用throw关键字，将该对象的引用抛出。

&ensp;&ensp;&ensp;&ensp;异常使我们将每件事都当做一个事务考虑，我们还可以将异常看作是一种内建的恢复系统，因为我们在程序中拥有不同的恢复点，如果程序的某部分失败了，异常将“恢复”到程序的某个已知点上。异常最大的好处就是在发生错误的时候能够强制程序终止当前线路的运行。
&ensp;&ensp;&ensp;&ensp;同其它对象的处理方式一样，我们使用new关键字在堆上创建异常对象，因此同样伴随着存储空间的分配和构造器的调用。所有标准异常器都有两个构造函数，一个是无参构造函数，另一个是可以传递一个字符串的构造函数，将相关信息放入到构造器中。此外，异常处理能够抛出任意类型的Throwable对象，它是异常类型的根类。通常对于不同类型的错误，要抛出对应的异常，错误信息可以保存在异常对象内部，或者使用异常的名称来暗示。

# 三、捕获异常

&ensp;&ensp;&ensp;&ensp;我们可以在程序中专门定义一个方法块儿，用来尝试各种可能产生异常的方法，这个块的定义形式如下：

```java
try{
    //---
}
```

&ensp;&ensp;&ensp;&ensp;在关键字try后包围的一部分代码块，称作try块，这样做比起之前说的要在每个会产生错误的地方进行判断要容易的多，并且代码的可读性大大增强，产生异常之后抛出的异常必须在某处得到处理，这个地点就是异常处理区，异常处理区紧跟着try块，使用catch关键字表示：

```
try{ //---}catch(Type1 arg1){ //---}catch(Type2 arg2){
//---
```

&ensp;&ensp;&ensp;&ensp;每个catch语句看起来像是一个接收一个且仅接收一个的特殊参数类型的方法。可以在方法块内部处理这个参数，当然有的异常见名知意，因此可以不用处理参数，但并不可以省略。可以有多个catch块与try对应，用来捕获多种不同的异常。异常处理程序必须紧跟在try块之后，当异常被抛出之后，异常处理机制负责搜寻参数与异常类型相匹配的第一个处理程序。然后进入catch块中执行，此时认为异常得到了处理。一旦catch子句结束，则认为处理程序的查找过程结束。只有匹配的catch字句才会执行，在try块内部不同的方法可能会产生相同的异常，而你只需要提供一个针对此类型的异常处理程序。异常的处理理论上有两种模型，一种是终止模型，表示程序发生异常之后，已经无法恢复到程序正常执行的顺序上，程序被迫发生终止。另一种模型是恢复模型，表示异常发生时，我们要做的是处理错误，而不是抛出异常。目前终止模型已经基本取代了恢复模型，虽然这种恢复模型看起来很好，但是实际上使用起来并不容易。

# 四、创建自定义异常

&ensp;&ensp;&ensp;&ensp;Java中虽然提供了很多默认的异常类型，但是要想完全覆盖会发生的异常情况显然是不现实的，因此我们可以自定义异常来表示我们预期可能会出现的异常。自定义的形式也非常简单，只需要继承一个相似的异常类即可。建立一个新的异常类最贱的方法就是让编译器为你产生默认构造器，所以这几乎不需要多少代码。

```java
package com.chenxyt.java.practice;  
class SimpleException extends Exception{}  
public class InheritingException {  
    public void f() throws SimpleException{  
        System.out.println("Throw SimpleException from f");  
        throw new SimpleException();  
    }  
    public static void main(String[] args) {  
        InheritingException ite = new InheritingException();  
        try{  
            ite.f();  
        }catch(SimpleException e){  
            System.out.println("Caught it!");  
        }  
    } 
}
```

运行结果：

![png1]([Java编程思想]十三：通过异常处理错误/png1.png)

&ensp;&ensp;&ensp;&ensp;当然我们还可以创建一个带有参数的构造器

```java
package com.chenxyt.java.practice;
 
class MyException extends Exception{
	public MyException(){
		
	}
	public MyException(String msg){
		super(msg);
	}
}
public class FullConstructors {
	public static void f() throws MyException {
		System.out.println("Throwing myException from f()");
		throw new MyException();
	}
	public static void g() throws MyException{
		System.out.println("Throwing myException from g()");
		throw new MyException("Originated in g()");
	}
	public static void main(String[] args) {
		try{
			f();
		}catch(MyException e){
			e.printStackTrace();
		}
		try{
			g();
		}catch(MyException e){
			e.printStackTrace(System.out);
		}
	}
}
```

运行结果：

![png2]([Java编程思想]十三：通过异常处理错误/png2.png)

&ensp;&ensp;&ensp;&ensp;在带参数的异常构造器中，显示的调用了其基类的带参构造方法。在catch块的异常处理程序中，调用了在Throwable类（所有异常的基类）声明的printStackTrace方法，该方法如果不带参数则将异常信息也就是从方法调用处到异常产生出的所有方法调用信息打印到标准错误流上，如果带了标准输出流参数，则打印在标准输出流上。区别可以从控制台错误信息的眼神看出。标准错误流更加的引人注目。

# 五、异常说明

&ensp;&ensp;&ensp;&ensp;Java提供了相应的语法（并强制使用这个语法），使你可以使用礼貌的方式告知客户端程序员某个方法可能会抛出的异常类型，然后客户端程序员就可以进行相应的处理。这就是异常说明，它属于方法声明的一部分，紧跟在形式参数列表之后，使用throws关键字，后面接一个所有潜在异常类型的列表，所以方法定义可能看起来像是这样：

```java
void f() throws TooBig,TooSmall,DivZero{};
```

&ensp;&ensp;&ensp;&ensp;代码必须与异常说明一致，比如上边的方法，如果我们在main（）中调用，编译器会提示该方法会产生异常，要么使用try catch进行处理，要么使用throws关键字抛出这种异常。

# 六、捕获所有异常

&ensp;&ensp;&ensp;&ensp; 我们可以使用所有异常类型的基类Exception来捕获所有的异常，即不管发生什么类型的异常，都能被捕获到。

```java
catch（Exception e）{
    //---
}
```

&ensp;&ensp;&ensp;&ensp;Exception是与编程相关的所有异常的基类（还有其它的基类），所以它不会具有太多的信息。不过可以调用它从其基类Throwable继承的方法

String getMessage（） 获取详细信息
String getLocalizedMessage（）获取用本地语言表示的详细信息
String toString（）返回对Throwable的简单描述
void printStackTrace（）打印Throwable的调用栈轨迹 输出到标准错误流
void printStackTrace（PrintStream）打印Throwable的调用栈轨迹 输出到可选择的流
void printStackTrace（PrintStream）打印Throwable的调用栈轨迹 输出到可选择的流
调用栈显示了“把你带到异常抛出地点”的方法调用序列。
&ensp;&ensp;&ensp;&ensp;Throwable fillInStackTrace（）用于在Throwable对象内部，记录栈帧的当前状态，这在程序重新抛出错误或者异常时很有必要。
&ensp;&ensp;&ensp;&ensp;此外，也可以使用Throwable从其基类Object继承的方法，getClass（）它将返回一个表示此类型的对象，然后可以使用getName（）方法查询这个Class对象包含包信息的名称，也可以使用getSimpleName（）返回只有类名的名称。
下面展示Exception类型方法的使用：

```java
package com.chenxyt.java.practice;
public class ExceptionMethods {
	public static void main(String[] args) {
		try{
			throw new Exception("My Exception");
		}catch(Exception e){
			System.out.println("Caught Exception");
			System.out.println("getMessage():" + e.getMessage());
			System.out.println("getLocalizedMessage():" + e.getLocalizedMessage());
			System.out.println("toString()" + e.toString());
			System.out.println("printStackTrace():--==");
			e.printStackTrace();
			System.out.println("printStackTrace(System.out):--==");
			e.printStackTrace(System.out);
		}
	}
}
```

运行结果：

![png3]([Java编程思想]十三：通过异常处理错误/png3.png)

&ensp;&ensp;&ensp;&ensp;printStackTrace（）方法所提供的信息可以使用getStackTrace（）方法获取到，这个方法返回一个由栈轨迹元素所构成的数组，其中每一个元素都表示栈中的一帧。元素0为栈顶元素，表示调用序列最后的一个方法，也就是Throwable创建和抛出的地方，数组中最后一个元素为方法调用序列中的第一个调用方法。下面程序简单演示：

```java
package com.chenxyt.java.practice;
public class WhoCalled {
	static void f(){
		try{
			throw new Exception();
		}catch(Exception e){
			for(StackTraceElement ste : e.getStackTrace()){
				System.out.println(ste.getMethodName());
			}
		}
	}
	static void g(){
		f();
	}
	static void h(){
		g();
	}
	public static void main(String[] args) {
		f();
		System.out.println("------------");
		g();
		System.out.println("------------");
		h();
	}
}
```

运行结果：

![png4]([Java编程思想]十三：通过异常处理错误/png4.png)

可以看到第一个打印的是异常抛出的方法，最后打印的是main（）函数方法。

&ensp;&ensp;&ensp;&ensp;有时候我们希望把刚捕获的异常抛出，如下：

```java
catch(Exception e){
    System.out.println("caught an exception");
    throw e;
}
```

&ensp;&ensp;&ensp;&ensp;重抛异常会把异常抛给上一级环境中的异常处理程序，同一个try块后边的catch块将被忽略。此外，异常对象的所有信息都会被保持，所以上一级环境的异常捕获程序可以获得 这个异常的所有信息。如果只是单纯的把这个异常抛出，那么printStackTrace（）显示的将是原来这个异常的栈调用信息。如果想更新这个序列信息，可以使用fillInStackTrace（）方法，这将返回一个Throwable对象，它是把当前调用栈的信息，填入到原来那个异常对象而建立的。如下：

```java
package com.chenxyt.java.practice;
public class ReThrowing {
	public static void f() throws Exception{
		System.out.println("the exception in f()");
		throw new Exception("exception from f()");
	}
	public static void g() throws Exception{
		try{
			f();
		}catch(Exception e){
			System.out.println("inside g e.printStackTrace");
			e.printStackTrace();
			throw e;
		}
	}
	public static void h() throws Exception{
		try{
			f();
		}catch(Exception e){
			System.out.println("inside h e.printStackTrace");
			e.printStackTrace();
			throw (Exception)e.fillInStackTrace();
		}
	}
	public static void main(String[] args) {
		try{
			g();
		}catch(Exception e){
			System.out.println("main:printStackTrace");
			e.printStackTrace();
		}
		try{
			h();
		}catch(Exception e){
			System.out.println("main:printStackTrace");
			e.printStackTrace();
		}
	}
}
```

运行结果：

![png5]([Java编程思想]十三：通过异常处理错误/png5.png)

&ensp;&ensp;&ensp;&ensp;可以看出当使用了fillInStackTrace（）方法之后，调用栈中的方法栈信息发生了变化。当然如果在捕获了第一种异常之后抛出了另一种异常，那么调用栈的信息也会发生变化，类似使用了fillInStackTrace（）方法。但是这样做的话，原始的异常信息就消失了，有时候我们希望在抛出新的异常的时候能保留原来的异常信息，这被称作是异常链。在JDK1.4之前，程序员需要自己编写代码保存原来的异常信息，现在所有的Throwable子类都可以在构造函数中传入一个cause参数，这个参数就用来保存原始异常。这样就可以在当前位置创建并抛出新的异常，也可以通过这个异常链追踪到原始异常。

&ensp;&ensp;&ensp;&ensp;现在在所有Throwable的子类中，只有三种基本异常类型提供了带有cause参数的构造器，分别是Error、Exception、RuntimeException，对于其他类型的异常链接，应该使用initCause（）方法，而不是构造器。

# 七、Java标准异常

&ensp;&ensp;&ensp;&ensp;Throwable这个类表示任何可以被作为异常抛出的类，Error表示编译时的系统错误，一般不需要我们关心。Exception是可以被抛出的基本类型，在Java类库、用户方法以及运行时故障都可以抛出此类异常。所以Java程序员通常需要关心此类的异常信息，同时Java异常要求可以见名知意。Java中还有一种Runtime异常，它表示程序编程错误，无法由程序员自己控制，这种异常不需要我们自己主动捕获并抛出，程序会自动抛出此类异常，如前文提到的NullPointerException。

# 八、使用finally进行清理

&ensp;&ensp;&ensp;&ensp;对于一些代码，可能希望无论是否抛出异常，这些代码都必须要执行。这通常包括内存回收之外的一些操作，因为内存回收是由垃圾回收器完成。为了达到这个效果，可以在catch块之后加上finally块。在finally块中处理这些代码。完整的异常处理如下：

```java
try{
	//---
}catch(Exception e1){
	//---
}catch(Exception e2){
	//---
}finally{
	//---
}
```

&ensp;&ensp;&ensp;&ensp;下面的示例用来证明finally字句不管异常是否抛出都能被执行：

```java
package com.chenxyt.java.practice;
public class FinallyWorks {
	static int count = 0;
	public static void main(String[] args) {
		while(true){
			try{
				if(count++ == 0){
					throw new Exception();
				}
				System.out.println("No Exception");
				}catch(Exception e){
					System.out.println("Exception");
				}finally{
					System.out.println("In finally clause");
					if(count == 2){
						break;
					}
				}
			}
		}
	}
```

&ensp;&ensp;&ensp;&ensp;可以看到不管异常是否被抛出，finally字句都执行了。这个程序告诉我们，当程序发生了异常之后不能正确的回到原来的执行顺序上，我们可以在finally块中处理一些必要做的事情。比如打开文件的操作，当文件操作发生异常的时候，我们要确保文件被正确的关闭。

# 九、异常的限制

&ensp;&ensp;&ensp;&ensp;当覆盖一个方法的时候，我们只能抛出在其基类方法异常说明列出的那些异常。这个限制的目的是符合面向对象的思想的，意味着其基类方法应用到派生类对象的时候，也能正常使用。异常也不例外。

```java
package com.chenxyt.java.practice;
class BaseballException extends Exception{
	
}
class Foul extends BaseballException{
	
}
class Strike extends BaseballException{
	
}
abstract class Inning{
	public Inning() throws BaseballException{
		
	}
	public void event() throws BaseballException{
		
	}
	public abstract void atBat() throws Strike,Foul;
	public void walk(){
		
	}
}
class StormException extends Exception{
	
}
class RainedOut extends StormException{
	
}
class PopFoul extends Foul{
	
}
interface Storm{
	public void events() throws RainedOut;
	public void rainHard() throws RainedOut;
}
public class StormyInning extends Inning implements Storm{
 
	public StormyInning() throws BaseballException {
		super();
		// TODO Auto-generated constructor stub
	}
 
	@Override
	public void rainHard() throws RainedOut {
		// TODO Auto-generated method stub
		
	}
 
	@Override
	public void atBat() throws Strike, Foul {
		// TODO Auto-generated method stub
		
	}
 
	@Override
	public void events() throws RainedOut {
		// TODO Auto-generated method stub
		
	}
	//抽象类中的该方法没有抛出任何异常 所以覆盖重写的时候 也不能抛出异常
//	public void walk() throws RainedOut{
//		
//	}
	//抽象类中的该方法抛出了异常 重写的时候可以选择不抛出异常
//	public void event(){
//		
//	}
	//抽象类中的该方法抛出了BaseballException 所以可以抛出其异常的子异常
	public void event() throws Foul{
		
	}
}
```

&ensp;&ensp;&ensp;&ensp;如注释上所述，当继承或者实现一个类的时候，这个子类只能抛出基类中异常列表中的异常，或者其子异常，或者不抛出异常，但是不能抛出其它异常，或者抛出异常的基类异常。

# 十、构造器

&ensp;&ensp;&ensp;&ensp;“如果异常发生了，所有东西能被正确清理吗？”大多数情况都是非常安全的，可以使用finally进行清理。但是当使用构造函数的时候，就有些问题了。比如使用构造函数构造文件操作，因为finally语句不管构造成功或者失败都会执行。假如构造失败了，如果文件没有被打开，那么这时候如果执行finally关闭文件操作就是不正确的。还有，如果文件构造成功了，因为文件需要在后边使用，所以在finally中关闭文件显然是不合理的。

```java
package com.chenxyt.java.practice;
import java.io.BufferedReader;
import java.io.FileNotFoundException;
import java.io.FileReader;
import java.io.IOException;
public class InputFile {
	private BufferedReader in;
	public InputFile(String name) throws Exception{
		try{
			in = new BufferedReader(new FileReader(name));
		}catch(FileNotFoundException e1){
			//没有找到文件也就是文件没有打开 所以不需要清理
			System.out.println("The file is not found");
			throw e1;
		}catch(Exception e2){
			//所有其它的异常都发生在文件打开之后 因此需要进行清理工作
			try{
				in.close();
			}catch(IOException e3){
				//---
			}
			throw e2;
		}finally{
			//此处由于构造成功也会执行 所以不能进行文件关闭操作
		}
	}
}
```

&ensp;&ensp;&ensp;&ensp;对于这种需要在构造器中创建的对象，合理的做法是使用嵌套try-catch

```java
package com.chenxyt.java.practice;
public class CleanUp {
	public static void main(String[] args) {
		try{
			InputFile in = new InputFile("CleanUp.java");
			try{
				//
			}catch(Exception e1){
				//此处为文件操作异常
			}finally{
				//-- 此处清理操作关闭文件
			}
		}catch(Exception e2){
			//--此处捕获构造异常
		}
	}
}
```

# 十一、异常匹配

&ensp;&ensp;&ensp;&ensp;在抛出异常之后，异常处理系统会按照代码的编写顺序找到最近的异常处理程序并执行，此时便任务异常已经得到了处理，就不会继续进行下去。查找的时候并不要求抛出的异常同处理程序所声明的异常完全匹配，子类的对象也可以匹配到其基类的处理程序。

```java
package com.chenxyt.java.practice;
class Annyoance extends Exception{
	
}
class Sneeze extends Annyoance{
	
}
public class Human {
	public static void main(String[] args) {
		try{
			throw new Sneeze();
		}catch(Sneeze s){
			System.out.println("Catch Sneeze");
		}catch(Annyoance a){
			System.out.println("Catch Annyoance1");
		}
		try{
			throw new Sneeze();
		}catch(Annyoance a){
			System.out.println("Catch Annyoance2");
		}
	}
}
```

运行结果：

![png6]([Java编程思想]十三：通过异常处理错误/png6.png)

&ensp;&ensp;&ensp;&ensp;编译器发现Annyoance是Sneeze的基类，所以编译的过程会提示警告。

# 十二、总结

&ensp;&ensp;&ensp;&ensp;异常是Java编程中不可或缺的一部分，它将程序正常执行的路径与错误处理分开，减少了编码判断的冗余。使用try-catch的方式进行异常处理，同时我们也可以创建自己的异常，继承其它的异常类。一些RuntimeException无需我们自己捕捉，程序会自动捕捉。注意构造器内发生异常的情况，因为构造器往往是只是简单的构造了一个对象，对象还需要使用，所以在使用finally进行清理的时候需要注意这一点。因为finally域的代码块不管异常是否发生都会执行。处理异常的时候可以使用基类Throwable提供的方法，比如使用printStackTrace打印栈的调用顺序。