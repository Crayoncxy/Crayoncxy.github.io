---
layout: posts
title: 【Java编程思想】十四：字符串
date: 2018-11-19 19:50:56
tags:
- Java 
- Think in Java
categories: 学习笔记
---

&ensp;&ensp;&ensp;&ensp;大量程序表明，字符串操作是程序设计中的最基本操作。

# 一、不可变String

&ensp;&ensp;&ensp;&ensp;String对象是不可变的，每一个看似修改了String值的方法，实际都是产生了一个新的字符串。

```java
package com.chenxyt.java.practice;
public class Immutable {
	public static String upCase(String s){
		return s.toUpperCase();
	}
	public static void main(String[] args) {
		String q = "howdy";
		System.out.println(q);
		String qq = upCase(q);
		System.out.println(qq);
		System.out.println(q);
	}
}
```

运行结果：

![png1]([Java编程思想]十四：字符串/png1.png)

&ensp;&ensp;&ensp;&ensp;当把q传给upCase方法时，实际上传递的是引用的一个拷贝。其实每当String对象作为参数传递时，传递的都是一个拷贝。再看upCase方法，只有当该方法运行时，局部引用s才存在，一旦该方法结束，引用s就消失了。该方法的返回值，实际上是最终值的引用，也就是upCase返回的引用已经指向了一个新的对象，而原本的q并没有发生任何变化。比如如下的方法，我们并不希望在经过一个操作之后，改变原有的对象。

```java
String s = "abcde";
String x = Immutable.upCase(s);
```

&ensp;&ensp;&ensp;&ensp;因为方法的参数是用来传递信息的，而不是用来改变原有对象本身的。

# 二、重载“+”与StringBuilder

&ensp;&ensp;&ensp;&ensp;String对象是不可变的，所以指向它的任何引用都不会改变该对象的值。不可变性会对效率带来一个问题，为String对象重载“+”操作符就是一个例子，重载的意思是一个操作符应用与特定的类时被赋有特殊的意义。（用于String的“+”和"+="是Java中仅有的两个重载过的操作符，Java不允许程序员自己重载操作符）
操作符“+”可以用来连接两个String

```java
package com.chenxyt.java.practice;
public class Concatetion {
	public static void main(String[] args) {
		String mango = "mango";
		String s = "abc" + mango + "def" + 42;
		System.out.println(s);
	}
}
```

运行结果：

![png2]([Java编程思想]十四：字符串/png2.png)

&ensp;&ensp;&ensp;&ensp;上述代码的运行过程可能是这样的：String可能有一个append方法，然后它会生成一个新的String对象用来连接abc和mango，然后该对象再与def相连生成新的对象，依次类推。这样做的话会产生很多中间垃圾需要清理因此它效率极低。
我们使用JDK自带的反编译工具查看上述代码如何工作：

![png3]([Java编程思想]十四：字符串/png3.png)

&ensp;&ensp;&ensp;&ensp;我们代码中并没有使用StringBuilder类，然而编译器却自动的引入了StringBuilder类，从编译后的代码可以看出，字符串连接工作主要的操作是编译器创建一个StringBuilder对象，然后调用该对象的append（）方法将所有的要连接的字符串连接到后边，最后调用toString（）方法转换成String对象存给s。
&ensp;&ensp;&ensp;&ensp;因此在编写一个类似toString（）的方法时，如果字符串较短时，我们可以使用普通的拼接方式，当字符串操作较为复杂的时候，我们在代码中直接创建一个StringBuilder对象进行操作效率会更加优异。

# 三、无意识的递归

&ensp;&ensp;&ensp;&ensp;我们希望使用toString（）方法打印出对象的内存地址，那么我们可能会考虑使用this关键字：

```java
package com.chenxyt.java.practice;
import java.util.ArrayList;
import java.util.List;
public class InfiniteRecursion {
	public String toString(){
		return "InfiniteRecursion address" + this;
	}
	public static void main(String[] args) {
		List<InfiniteRecursion> v = new ArrayList<InfiniteRecursion>();
		for(int i = 0;i<10;i++){
			v.add(new InfiniteRecursion());
		}
		System.out.println(v);
	}
}
```

运行结果：

![png4]([Java编程思想]十四：字符串/png4.png)

这里当运行

```java
return "InfiniteRecursion address" + this;
```

时发生了类型转换，编译器发现“+”后边不是String类型，会试图转换成String类型，转换的方式就是调用toString方法，因此会递归调用，此处如果想正确打印地址，那么需要使用其基类Object的toString（）方法。

# 四、String上的操作

&ensp;&ensp;&ensp;&ensp;所有的类都是Class类的对象，使用Class类的getMethods（）方法可以返回该类的所有方法：

```java
package com.chenxyt.java.practice;
import java.lang.reflect.Method;
public class StringMethods {
	public static void main(String[] args) {
		Class<String> c = String.class;
		Method[] methods = c.getMethods();
		for(Method method :methods){
			System.out.println(method);
		}
	}
}
```

&ensp;&ensp;&ensp;&ensp;运行结果：

```java
public int java.lang.String.hashCode()
//比较两个字符串 重载了Object的方法 当两个字符串值相同、类型相同则认为相同 忽略了引用
public boolean java.lang.String.equals(java.lang.Object)
public java.lang.String java.lang.String.toString()
//获取指定索引下标位置上的字符
public char java.lang.String.charAt(int)
public int java.lang.String.codePointAt(int)
public int java.lang.String.codePointBefore(int)
public int java.lang.String.codePointCount(int,int)
public int java.lang.String.compareTo(java.lang.Object)
//按词典顺序比较两个字符串 大小写并不等价
public int java.lang.String.compareTo(java.lang.String)
public int java.lang.String.compareToIgnoreCase(java.lang.String)
//字符串连接 返回一个新的String为指定String连接参数
public java.lang.String java.lang.String.concat(java.lang.String)
//查找字符串中是否包含指定字符 存在返回true
public boolean java.lang.String.contains(java.lang.CharSequence)
//比较两个字符串
public boolean java.lang.String.contentEquals(java.lang.StringBuffer)
public boolean java.lang.String.contentEquals(java.lang.CharSequence)
public static java.lang.String java.lang.String.copyValueOf(char[])
public static java.lang.String java.lang.String.copyValueOf(char[],int,int)
public boolean java.lang.String.endsWith(java.lang.String)
//比较字符串是否相同 忽略大小写
public boolean java.lang.String.equalsIgnoreCase(java.lang.String)
public static java.lang.String java.lang.String.format(java.lang.String,java.lang.Object[])
public static java.lang.String java.lang.String.format(java.util.Locale,java.lang.String,java.lang.Object[])
public void java.lang.String.getBytes(int,int,byte[],int)
public byte[] java.lang.String.getBytes(java.lang.String) throws java.io.UnsupportedEncodingException
public byte[] java.lang.String.getBytes(java.nio.charset.Charset)
public byte[] java.lang.String.getBytes()
// s.getChars(1,2,char,3) 赋值s串中下标为1-2的到char中 char中起始位置为3
public void java.lang.String.getChars(int,int,char[],int)
public int java.lang.String.indexOf(int,int)
public int java.lang.String.indexOf(java.lang.String,int)
//是否包含该字符 包含返回下标 否则返回-1
public int java.lang.String.indexOf(java.lang.String)
public int java.lang.String.indexOf(int)
public native java.lang.String java.lang.String.intern()
public boolean java.lang.String.isEmpty()
public int java.lang.String.lastIndexOf(int,int)
public int java.lang.String.lastIndexOf(java.lang.String)
public int java.lang.String.lastIndexOf(java.lang.String,int)
public int java.lang.String.lastIndexOf(int)
//String 中字符的个数
public int java.lang.String.length()
public boolean java.lang.String.matches(java.lang.String)
public int java.lang.String.offsetByCodePoints(int,int)
public boolean java.lang.String.regionMatches(boolean,int,java.lang.String,int,int)
public boolean java.lang.String.regionMatches(int,java.lang.String,int,int)
//把字符串中的第一个参数字符替换成第二个
public java.lang.String java.lang.String.replace(char,char)
public java.lang.String java.lang.String.replace(java.lang.CharSequence,java.lang.CharSequence)
public java.lang.String java.lang.String.replaceAll(java.lang.String,java.lang.String)
public java.lang.String java.lang.String.replaceFirst(java.lang.String,java.lang.String)
public java.lang.String[] java.lang.String.split(java.lang.String,int)
public java.lang.String[] java.lang.String.split(java.lang.String)
public boolean java.lang.String.startsWith(java.lang.String,int)
//字符串起始串
public boolean java.lang.String.startsWith(java.lang.String)
public java.lang.CharSequence java.lang.String.subSequence(int,int)
//字符串截断
public java.lang.String java.lang.String.substring(int)
public java.lang.String java.lang.String.substring(int,int)
//返回一个字符数组，该字符数组包含字符串的所有字符
public char[] java.lang.String.toCharArray()
//字符串字符转换成小写
public java.lang.String java.lang.String.toLowerCase()
public java.lang.String java.lang.String.toLowerCase(java.util.Locale)
//字符串字符转换成大写
public java.lang.String java.lang.String.toUpperCase()
public java.lang.String java.lang.String.toUpperCase(java.util.Locale)
//删除String两端的空白字符
public java.lang.String java.lang.String.trim()
public static java.lang.String java.lang.String.valueOf(char)
public static java.lang.String java.lang.String.valueOf(float)
public static java.lang.String java.lang.String.valueOf(int)
public static java.lang.String java.lang.String.valueOf(long)
public static java.lang.String java.lang.String.valueOf(double)
public static java.lang.String java.lang.String.valueOf(java.lang.Object)
public static java.lang.String java.lang.String.valueOf(char[])
public static java.lang.String java.lang.String.valueOf(char[],int,int)
public static java.lang.String java.lang.String.valueOf(boolean)
public final native java.lang.Class java.lang.Object.getClass()
public final native void java.lang.Object.notify()
public final native void java.lang.Object.notifyAll()
public final void java.lang.Object.wait(long,int) throws java.lang.InterruptedException
public final native void java.lang.Object.wait(long) throws java.lang.InterruptedException
public final void java.lang.Object.wait() throws java.lang.InterruptedException
```

注释了一些常用的方法，其它的方法可自行查阅。

# 五、格式化输出

&ensp;&ensp;&ensp;&ensp;JavaSE5提供了格式化输出功能，这一功能使得控制输出的功能变得更加简单，同时也给开发者带来了更加强大的代码输出控制能力。JavaSE5引入的format方法可以用于PrintStream或PrintWriter对象，其中也包括System.out对象。format（）方法模仿在C语言的printf（）如下简单示例：

```java
package com.chenxyt.java.practice;
public class SimpleFormat {
	public static void main(String[] args) {
		int x = 5;
		double y = 5.333221;
		System.out.println("Row1: [" + x + " " + y + "]");
		System.out.format("Row1: [%d %f]\n",x,y);
		System.out.printf("Row1: [%d %f]\n",x,y);
	}
}
```

运行结果：

![png5]([Java编程思想]十四：字符串/png5.png)

可以看到，format（）与printf（）相同，只需要加上对应的格式化字符即可。

&ensp;&ensp;&ensp;&ensp;Java中所有新的格式化功能都由java.util.Formatter类处理，可以将其看做是一个翻译器，它将你的格式化字符串和数据翻译成想要的结果。当你创建了一个Formatter对象的时候，需要向编译器传递一些信息，告诉他最终的结果将向哪里输出。

```java
package com.chenxyt.java.practice;
 
import java.io.PrintStream;
import java.util.Formatter;
public class Tutle {
	private String name;
	private Formatter f;
	public Tutle(String name,Formatter f){
		this.f = f;
		this.name = name;
	}
	public void move(int x,int y){
		f.format("%s The Tutle is at (%d,%d)\n",name,x,y);
	}
	public static void main(String[] args) {
		Tutle tommy = new Tutle("Tommy",new Formatter(System.out));
		Tutle terry = new Tutle("Terry",new Formatter(System.err));
		tommy.move(0, 4);
		terry.move(4, 8);
	}
}
```

运行结果：

![png6]([Java编程思想]十四：字符串/png6.png)

如上我们指定了格式化输出的结果分别打印到了System.out和System.err上

&ensp;&ensp;&ensp;&ensp;有的时候我们希望做一些更精致的格式化信息，比如控制空格与对齐，最常见的是控制域的最小尺寸，这可以通过指定width实现，Formatter对象通过在必要时添加空格，来确保一个域至少达到某个长度。默认情况下域是右对齐的，不过也可以通过“-”指定数据的对齐方式。

```java
package com.chenxyt.java.practice;
import java.util.Formatter;
public class Reciept {
	private double total = 0;
	private Formatter f = new Formatter(System.out);
	public void printTitle(){
		f.format("%-15s %5s %10s\n","Item","Qty","Price");
		f.format("%-15s %5s %10s\n","----","---","-----");
	}
	public void print(String name,int qty,double price){
		f.format("%-15.15s %5d %10.2f\n",name,qty,price);
		total+=price;
	}
	public void printTotal(){
		f.format("%-15.15s %5s %10.2f\n","Tax","",total*0.06);
		f.format("%-15.15s %5s %10s\n","","","------");
		f.format("%-15.15s %5s %10.2f\n","Total","",total*1.06);
	}
	public static void main(String[] args) {
		Reciept receipt = new Reciept();
		receipt.printTitle();
		receipt.print("Jack's Magic Beans",4,4.25);
		receipt.print("Princess Beans",3,5.1);
		receipt.print("Thres Bears Porridge",1,14.25);
		receipt.printTotal();
	}
}
```

运行结果：

![png7]([Java编程思想]十四：字符串/png7.png)

可以看到Formatter类提供了很强大的格式化支持

Formatter类也有很多类型转换，常用的类型转换如下

d：整数型（十进制）e：浮点数（科学计数）c：Unicode字符 x：整数（十六进制）b：Boolean值 h：散列码（十六进制）s：String %：字符“%”f：浮点数（十进制），针对不同的数据类型，有些转换是无效的，如果强制使用则会发生异常。

# 六、正则表达式

&ensp;&ensp;&ensp;&ensp;正则表达式就是以某种方式来描述字符串，因此你可以说“如果一个字符串含有某些东西，那么它就是我需要的东西”。\d表示一位数字，Java中对于\反斜线有不同的处理，在其它语言中，\\表示我想插入一个普通的反斜线，而在Java中\\表示我将要插入一个正则表达式反斜线，所以它后边的值应该具有特殊意义。例如你想表示一个数字，那么就是\\d，如果想插入一个普通的反斜线，则需要使用\\\\，不过制表之类的应该使用单反斜线，\d\n，使用？表示可能存在，使用+表示一个或多个+之前的表达式。比如我们要判断可能有一个负号后边跟着一个或多个数字则使用：

-？\\d+

应用正则的最简单方式是使用String类提供的方法：

```java
package com.chenxyt.java.practice;
public class IntegerMatch {
	public static void main(String[] args) {
		System.out.println("-1234".matches("-?\\d+"));
		System.out.println("5678".matches("-?\\d+"));
		System.out.println("+911".matches("-?\\d+"));
		System.out.println("+911".matches("(-|\\+)?\\d+"));
	}
}
```

运行结果：

![png8]([Java编程思想]十四：字符串/png8.png)

&ensp;&ensp;&ensp;&ensp;前两个满足正则表达式的结果，第三个“+“作为一个独立的符号，所以与”可能有个负号开头的一个或多个整数“不匹配，应该使用”可能有一个正号或者符号开头的一个或多个整数“，括号有着分组的作用，|表示或。

&ensp;&ensp;&ensp;&ensp;String类还带有一个非常有用的正则表达式方法split（），功能是将字符串在正则表达式匹配的地方分割开：

```java
package com.chenxyt.java.practice; import java.util.Arrays; public class Splitting { public static String knights = "Then。when you have found the shrubbery.you must" + "cut down the mightiest tree in the forest。。。" + "with... a herring"; public static void split(String regex){ System.out.println(Arrays.toString(knights.split(regex))); } public static void main(String[] args) { split(" "); split("\\W+");
split("\\w+");
split("n\\W+"); } }
```

运行结果：

![png9]([Java编程思想]十四：字符串/png9.png)

&ensp;&ensp;&ensp;&ensp;第一个是使用空格拆分，\\W表示非单词字符，\\w表示单词字符，第三个表示字母n以及后边的一个或多个非单词字符。可见与正则表达式相同的内容都消失了。

&ensp;&ensp;&ensp;&ensp;String类自带的最后一个正则表达式方法为替换，你可以替换正则表达式第一个匹配的子串，也可以替换所有匹配的地方。

```java
public class Replacing {  
    static String s = Splitting.knights;  
    public static void main(String[] args) {  
        System.out.println(s.replaceFirst("f\\w+","located"));  
        System.out.println(s.replaceAll("shrubbery|tree|herring","banana"));  
    }  
}
```

运行结果：

![png10]([Java编程思想]十四：字符串/png10.png)

&ensp;&ensp;&ensp;&ensp;第一个正则表示以字母f开头后面跟着一个或多个字母的，将第一个匹配的子串替换成“located”，第二个表达式是替换三个单词中的任意一个，只要存在则替换。

&ensp;&ensp;&ensp;&ensp;我们也可以自己创建正则表达式，java.util.regex.Pattern类提供了完整的正则表达式列表，先列举一些常用的：

![png11]([Java编程思想]十四：字符串/png11.png)

&ensp;&ensp;&ensp;&ensp;以下是一些字符类创建的典型方式，以及一些预定义类。

![png12]([Java编程思想]十四：字符串/png12.png)

作为演示，下面的每一个正则表达式都可以成功的匹配“Rudolph“

```java
package com.chenxyt.java.practice;
public class Rudolph {
	public static void main(String[] args) {
		for(String parten:new String[]{"Rudolph","[rR]udolph","[rR][aeiou][a-z]ol.*","R.*"}){
			System.out.println("Rudolph".matches(parten));
		}
	}
}
```

运行结果：

![png13]([Java编程思想]十四：字符串/png13.png)

&ensp;&ensp;&ensp;&ensp;我们的任务是在能够完成任务的情况下编写简单的易于理解的正则表达式，而不是编写难以理解的正则表达式。
我们在使用正则表达式的时候很容易混淆，因为他是一种在Java之上的新语言，CharSequence，该接口从CharBuffer、String、StringBuffer、StringBuilder类之中抽象出了字符序列的一般化定义。

```java
package com.chenxyt.java.practice;
public interface CharSequence {
	char charAt(int i );
	int length();
	String subSequence(int start,int end);
	String toString();
}
```

因此，这些类都实现了该接口，多数正则表达式操作都接受CharSequence类型的参数。

&ensp;&ensp;&ensp;&ensp;String类作为正则表达式的匹配使用功能有限，因此我们可以使用更为强大的正则表达式对象。只需要导入java.util.regex包，然后使用Pattern.compile（）方法编译正则表达式。它会根据传入的String类型的正则表达式生成一个Pattern对象，接下来把我们想要检索的字符串传入Pattern.matcher（）方法，该方法会生成一个Matcher对象，它有很多功能可以使用。
以下为演示示例，第一个控制台参数为要匹配的字符串，后边一个或多个参数都为正则表达式：

```java
package com.chenxyt.java.practice;
import java.util.regex.Matcher;
import java.util.regex.Pattern;
public class TestRegularExpression {
	public static void main(String[] args) {
		if(args.length<2){
			System.out.println("Usage:\n java TestRegularExpression " + "characterSequence regularExpression+");
			System.exit(0);
		}
		System.out.println("Input: \"" + args[0] + "\"");
		for(String arg:args){
			System.out.println("Regular expression: \"" + arg + "\"");
			//编译正则表达式
			Pattern p = Pattern.compile(arg);
			//传入字符串进行匹配
			Matcher m = p.matcher(args[0]);
			while(m.find()){
				System.out.println("Match \"" + m.group() + "\" at positions " + m.start() + "-" + (m.end() - 1));
			}
		}
	}
}
```

运行结果：

![png14]([Java编程思想]十四：字符串/png14.png)

&ensp;&ensp;&ensp;&ensp;Matcher.find（）方法用来查找字符串中的多个匹配。start（）方法返回匹配的起始位置的索引，end（）方法匹配的终止位置的索引。

# 七、扫描输入

&ensp;&ensp;&ensp;&ensp;目前来说，读取文本或者从标准输入读取数据一般的解决方法是读入一行文本，然后对其进行分词，然后使用Integer、Double的各种解析方法来解析数据。

```java
package com.chenxyt.java.practice;
 
import java.io.BufferedReader;
import java.io.IOException;
import java.io.StringReader;
 
public class SimpleRead {
	public static BufferedReader input = new BufferedReader(new StringReader("Sir Robin of Camelot\n22 1.523222"));
	public static void main(String[] args) {
		try{
			System.out.println("What's your name?");
			String name = input.readLine();
			System.out.println(name);
			System.out.println("How old are you?What's your favorite double");
			String numbers = input.readLine();
			System.out.println(numbers);
			String numArray[] = numbers.split(" ");
			int age = Integer.parseInt(numArray[0]);
			double favorite = Double.parseDouble(numArray[1]);
			System.out.format("Hi %s.\n",name);
			System.out.format("In 5 years you will be %d. \n",age+5);
			System.out.format("My Favorite double is %f.",favorite/2);
			}catch(IOException e){
				System.err.println("I/O Exception");
			}
	}
}
```

运行结果：

![png15]([Java编程思想]十四：字符串/png15.png)

&ensp;&ensp;&ensp;&ensp;以上代码中，使用了IO中的readLine（）方法读取输入流中的一行，然后进行解析。终于在JavaSE5新增了Scanner类，它可以大大的减轻扫描输入的工作。

```java
package com.chenxyt.java.practice;
import java.util.Scanner;
public class BetterRead {
	public static void main(String[] args) {
		Scanner stdin = new Scanner(SimpleRead.input);
		System.out.println("What's your name?");
		String name = stdin.nextLine();
		System.out.println(name);
		System.out.println("How old are you?What's your favorite double");
		System.out.println("(input:<age><double>)");
		int age = stdin.nextInt();
		double favorite = stdin.nextDouble();
		System.out.println(age);
		System.out.println(favorite);
		System.out.format("Hi %s.\n",name);
		System.out.format("In 5 years you will be %d. \n",age+5);
		System.out.format("My Favorite double is %f.",favorite/2);
	}
}

```

运行结果：

![png16]([Java编程思想]十四：字符串/png16.png)

&ensp;&ensp;&ensp;&ensp;Scanner的构造器可以接受任意类型的输入对象，其普通的next方法将返回一个String，而所有的基本类型都有一个next方法，该方法的作用是，读取到下一个完整的指定类型的分词之后才返回。同时Scanner还有对应的hasNext方法，用来判断是否有所输入分词的类型。
Scanner默认的定界符为空白符，我们也可以自己使用正则表达式指定分隔符：

```java
package com.chenxyt.java.practice;
import java.util.Scanner;
public class ScannerDelimiter {
	public static void main(String[] args) {
		Scanner scanner = new Scanner("12,24    ,35,67");
		scanner.useDelimiter("\\s*,\\s*");
		while(scanner.hasNextInt()){
			System.out.println(scanner.nextInt());
		}
	}
}
```

运行结果：

![png17]([Java编程思想]十四：字符串/png17.png)

&ensp;&ensp;&ensp;&ensp;这个例子使用逗号以及任意的空白字符来实现字符串的分割。使用useDelimiter（）方法设置定界符，也可以使用delimiter（）方法返回现在所使用的定界符。

&ensp;&ensp;&ensp;&ensp;Scanner除了可以扫描基本类型外，还可以扫描正则表达式，这在扫描复杂数据类型的时候非常有用：

```java
package com.chenxyt.java.practice;
 
import java.util.Scanner;
import java.util.regex.MatchResult;
 
public class ThreadAnalyzer {
	static String threatData = "58.27.82.161@02/10/2005\n" +
								"204.45.234.40@02/11/2005\n" +
								"58.27.82.161@02/11/2005\n" +
								"58.27.82.161@02/11/2005\n" +
								"58.27.82.161@02/11/2005\n" +
								"[Next log section width different data format]";
	public static void main(String[] args) {
		Scanner scanner = new Scanner(threatData);
		String pattern = "(\\d+[.]\\d+[.]\\d+[.]\\d+)@" + "(\\d{2}/\\d{2}/\\d{4})";
		while(scanner.hasNext(pattern)){
			scanner.next(pattern);
			MatchResult match = scanner.match();
			String ip = match.group(1);
			String date = match.group(2);
			System.out.format("Thread on %s from %s\n",date,ip);
			
		}
	}
	
}
```

运行结果：

![png18]([Java编程思想]十四：字符串/png18.png)

&ensp;&ensp;&ensp;&ensp;当next（）方法配合指定正则表达式使用时，将找到下一个匹配该模式的输入部分，调用match（）方法获得匹配的结果。有一点需要注意，它只针对下一个输入分词进行匹配，如果正则表达式含有定界符，那将永远不能成功。

# 八、StringTokenizer

&ensp;&ensp;&ensp;&ensp;在正则表达式和Scanner引入之前，我们使用的是StringTokenizer来进行字符串匹配，下面演示这种方式并与其它方式进行比较：

```java
package com.chenxyt.java.practice;
 
import java.util.Arrays;
import java.util.Scanner;
import java.util.StringTokenizer;
 
public class ReplacingStringTokenizer {
	public static void main(String[] args) {
		String input = "But I'm not dead yet!I feel happy!";
		StringTokenizer stoke = new StringTokenizer(input);
		while(stoke.hasMoreElements()){
			System.out.print(stoke.nextToken() + " ");
		}
		System.out.println();
			System.out.println(Arrays.toString(input.split(" ")));
			Scanner scanner = new Scanner(input);
			while(scanner.hasNext()){
				System.out.print(scanner.next() + " ");
			}
	
	}
}
```

运行结果：

![png19]([Java编程思想]十四：字符串/png19.png)

&ensp;&ensp;&ensp;&ensp;使用正则表达式或者Scanner我们可以使用更加复杂的模式匹配字符串，而使用StringTokenizer则很困难了，因此实际上StringTokenizer这个类基本上已经被废弃了。

# 九、总结

&ensp;&ensp;&ensp;&ensp;String类型作为程序设计中最为常见的一种操作类型，它是一种不可变的操作类型。它内部重载了“+”操作符，使得可以使用"+"操作符完成字符串的拼接工作，拼接的原理是编译器为我们自动生成了StringBuilder类来完成。因此对于复杂的字符串拼接操作，我们自己使用StringBuilder效率会更高一些。String还有很多常用的方法，正则表达式为我们提供了字符串匹配的多种形式。