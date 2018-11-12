---
title: 【Java编程思想】四：操作符
date: 2018-11-12 10:30:31
tags:
- Java 
- Think in Java
categories: 学习笔记
---

# 一、更简单的打印语句

&ensp;&ensp;&ensp;&ensp;在第二章中使用了一个打印语句：

```java
System.out.println("Hello World");
```

&ensp;&ensp;&ensp;&ensp;结合static的静态包用法，如果这个语句在多个地方进行调用，那么可以将这个语句写成静态方法，然后通过导入静态包的形式使用静态方法：

test包下的Printer类：

```java
package com.chenxyt.java.test;
public class Printer {
	public static void print(String msg){
		System.out.println(msg);
	}
}
```

practice包下的TestStatic类：

```java
package com.chenxyt.java.practice;
import static com.chenxyt.java.test.Printer.*;
public class TestStatic{
	public static void main(String[] args) {
		print("This is TestStatic");
	}
}
```

# 二、使用Java操作符

&ensp;&ensp;&ensp;&ensp;Java与其他语言一样，支持加号，正号，减号，负号，乘除号等操作符，同时“=”，“==”和“！=”不光可以操作基本操作类型，还可以操作所有的对象。此外，String类支持“+”和“+=”。

# 三、优先级

&ensp;&ensp;&ensp;&ensp;当一个表达式中存在多个操作符时，操作符的优先级就尤为重要了，它决定了程序运算操作执行的先后顺序。Java中的计算顺序与其它语言的基本相同，先计算乘除，再计算加减，有括号的先计算括号里边的。System.out.println()语句中包含“+”的操作符号，简单的只是进行字符串的连接，复杂一点的就是当编译器发现“+”前边是一个String类型，会尝试将“+”后面的内容转换成String类型。

# 四、赋值

&ensp;&ensp;&ensp;&ensp;赋值操作使用的是“=”操作符，将“=”右边的值赋给左边，右值可以是任意的常数、变量或者是表达式，只有它能生成一个值即可，而左值必须是一个明确的已命名的变量，就是必须要有个物理空间来进行存储。比如，可以将一个常数赋给变量：

```java
a=4;
```

但是却不能将一个变量赋值给一个常数:

```java
4=a;
```

&ensp;&ensp;&ensp;&ensp;对于基本的数据类型，赋值操作没有引用的涉及，只是单纯的将一个值赋值给另一个值。例如a=b，当b再次被修改时，a不会受到影响，因为a与b相互独立。但是对于对对象的赋值来说，情况却大大不同，因为我们对对象的操作是操作了对象的引用，所以当一个对象赋值给另一个对象时，实际上是拷贝了引用到左值，也就是比如c和d是指向两个不同对象的引用，当c=d时，实际发生的情况是c和d都指向了原本只有d指向的对象。而c被赋值之后，原来的引用丢失了，它曾经所指向的不再被引用的对象被垃圾回收器回收了。如下例子创建了两个不同的对象，进行赋值操作。

```java
package com.chenxyt.java.practice;
class Tank{
	int level;
}
public class CopyTest{
	public static void main(String[] args) {
		Tank tk1 = new Tank();
		Tank tk2 = new Tank();
		tk1.level = 2;
		tk2.level = 3;
		System.out.println("tk1.level = " + tk1.level + "---tk2.level = " + tk2.level);
		tk1=tk2;
		System.out.println("tk1.level = " + tk1.level + "---tk2.level = " + tk2.level);
		tk2.level=5;
		System.out.println("tk1.level = " + tk1.level + "---tk2.level = " + tk2.level);
	}
}
```

运行结果如下：

![png1]([Java编程思想]四：操作符/png1.png)

&ensp;&ensp;&ensp;&ensp;tk1和tk2分别是两个独立的对象，它们内部有level属性，赋值之前两个对象的属性值不同，赋值操作完成之后两个对象的属性值相同，当再次更改对象tk2的level值时，预期的理想情况是不会影响tk1的值，但实际结果并非如此，tk1与tk2对象的属性相同，这与前面的分析结果相同。
&ensp;&ensp;&ensp;&ensp;Java中这种针对对象的特殊现象叫做“**别名现象**”，如果想避免这种现象的话，可以使用如下操作，赋值操作针对属性而不是对象的引用:

```java
tk1.level=tk2.level
```

这样操作就可以保持两个对象本身相互独立。

&ensp;&ensp;&ensp;&ensp;在调用方法传参的时候也是会产生“别名问题”，如下例子，调用方法copy之后，理想的是只改变了方法内的值，实际上是改变了方法之外对象的值：

```java
package com.chenxyt.java.practice;
class Tank{
	int level;
}
public class CopyTest{
	public static void copy(Tank tk3){
		tk3.level = 9;
	}
	public static void main(String[] args) {
		Tank tk1 = new Tank();
		tk1.level = 2;
		System.out.println("tk1.level = " + tk1.level);
		copy(tk1);
		System.out.println("tk1.level = " + tk1.level);
	}
}
```

运行结果，方法外部tk1对象的值被修改了

![png2]([Java编程思想]四：操作符/png2.png)

# 五、算术操作符

&ensp;&ensp;&ensp;&ensp;Java中的算术操作符与其它语言基本类似，有加号（+）、减号（-）、乘号（*）、取整（/）、取余（%），同时也具有简化运算符的功能如要将x加4之后再赋值给x，则可以写成：

```java
x+=4;
```

# 六、自动递增递减

&ensp;&ensp;&ensp;&ensp;Java中的递增递减操作与其它语言基本类似，递增符合（++），递减符合（--），操作目的是快速使一个整数加1或者减1如a++等同于a=a+1，这两种符号分别有两种使用方式称为“前缀式”和“后缀式”，前缀式意味着符号在变量前边，后缀式意味着符号在变量后边。二者的区别，对于前缀式是先做运算再取值，如a=1，b=++a，此时a跟b的值都是2，而后缀式则是先取值再做运算，如a=1，b=a++，此时b的值为1，a的值为2

# 七、关系操作符

&ensp;&ensp;&ensp;&ensp;关系操作符生成的是一个boolean（布尔）结果，它们计算操作数之间的关系，如果关系为真则结果为true，如果关系为假则结果为false。关系操作符号包括小于（＜）、大于（＞）、小于等于（＜=）、大于等于（＞＝）、等于（==）以及不等于（！=）。等于和不等于适用于所有的基本数据类型，而其它的操作符不适用于boolean类型，因为它们的值为true或者false，比较大小没有意义。
操作符“==”和“！=”同样适用于操作对象，但是与基本操作类型相比有一些不同。

```java
package com.chenxyt.java.practice;
public class OperationTest{
	public static void main(String[] args) {
		Integer itg1 = new Integer(22);
		Integer itg2 = new Integer(22);
		System.out.println(itg1==itg2);
		System.out.println(itg1!=itg2);
		int int1 = 22;
		int int2 = 22;
		System.out.println(int1==int2);
		System.out.println(int1!=int2);
	}
}
```

如上创建了两个Integer对象和两个int基本数据类型值，分别做“==”和“！=”判断，结果如下：

![png3]([Java编程思想]四：操作符/png3.png)

&ensp;&ensp;&ensp;&ensp;从结果可见，两个对象是“！=”，而两个int基本数据类型值是“==”，这是为什么呢？因为对于对象来说，“==”和“！=”比较的是对象的引用，虽然这两个对象的值相同，但是他们对象的引用并不是一个，也就是他们在内存中有两个不同的存储位置，所以不相同。所以要想比较对象的值是否相同，我们可以使用对象的equals（）方法来进行比较，它是Object类的一个通用方法，第一章中说到过所有类的父类都是Object，因此任何类的对象都可以调用这个方法。

```java
package com.chenxyt.java.practice;
public class OperationTest{
	public static void main(String[] args) {
		Integer itg1 = new Integer(22);
		Integer itg2 = new Integer(22);
		System.out.println(itg1.equals(itg2));
	}
}
```

结果如下：

![png4]([Java编程思想]四：操作符/png4.png)

&ensp;&ensp;&ensp;&ensp;新的问题来了，我们看如下示例，我们自定义了一个新的类，创建了两个对象进行比较：

```java
package com.chenxyt.java.practice;
class Oper{
	int i;
}
public class OperationTest{
	public static void main(String[] args) {
		Oper op1 = new Oper();
		Oper op2 = new Oper();
		op1.i=2;
		op2.i=2;
		System.out.println(op1.equals(op2));
	}
}
```

结果如下：

![png5]([Java编程思想]四：操作符/png5.png)

&ensp;&ensp;&ensp;&ensp;结果仍然为fasle！这是为什么呢？这是因为Object类中equals（）方法实际上是比较两个引用是否相同，就是它与“==”的本质效果是一样的，但是在第一个示例中的Integer类中，Java覆盖了这个方法，方法内容判断两个对象的类型是否相同以及值是否相同即可。而我们自己创建的Oper类并没有覆盖这个方法，所以沿用的还是Object类的方法。常见的String类也是覆盖了equals（）方法，效果与Integer类的对象相同。

# 八、逻辑操作符

&ensp;&ensp;&ensp;&ensp;Java中的逻辑操作符与其它语言基本类似，包括与（&&）、或（||）、非（！），不同的是Java中的逻辑操作符只能应用与布尔值之间或者是表达式结果为布尔值。

```java
package com.chenxyt.java.practice;
public class OperationTest{
	public static void main(String[] args) {
		int a = 1;
		int b = 2;
		int c = 3;
		System.out.println((a>b)&&(b>c));
		System.out.println((a>b)||(b<c));
		System.out.println(!(a<b));		
	}
}
```

运行结果如下：

![png6]([Java编程思想]四：操作符/png6.png)

&ensp;&ensp;&ensp;&ensp;逻辑运算满足如下关系，“&&”逻辑与运算符必须两边同时为真则结果为真，否则为假。“||”逻辑或运算符只要有一边为真则结果为真，“！”非运算符取对应值的对立面，即非真则假。对于逻辑与和逻辑或运算还有个名词叫做**短路**，即当两个算式做逻辑与运算时，如果左边第一个算式为假，则显然这个逻辑表达式最后的结果一定为假，那么就没有必要进行下边的表达式计算，直接结束运算，这个过程称作短路。

# 九、直接常量

&ensp;&ensp;&ensp;&ensp;Java中同样可以使用例如“π”这种常量表示。

# 十、按位操作符

&ensp;&ensp;&ensp;&ensp;Java中的按位操作符与其他语言基本相似，有按位与（&）、按位或（|）、按位异或（^）和按位非（~）操作符。它们针对基本数据类型的一个比特位（bit）进行运算。运算规则如下，&操作符必须同时为1才为1，|操作符只有同时为0时才为0，^操作符只要有一个为1就为1，~是单目运算符，取反，若值为1则结果为0，反之为1。按位操作符除了~还可以与“=”合起来使用，如&=或者|=、^=。
对于布尔值，同样可以进行按位与、按位或和按位异或运算，但是不能进行按位非运算，并且他们不会被短路，不管第一个表达式结果是什么，都会继续运算下去。

# 十一、移位操作符

&ensp;&ensp;&ensp;&ensp;移位操作符操作的也是二进制的“位”，移位操作符只能用来处理整数类型。移位操作也是二元操作符，一共有三种，左移操作符（<<），右移操作符（>>），无符号右移操作符（>>>），左移操作符是操作符左边的数向左移动操作符右边指定的位数，**低位补0**，右移操作符是操作符左边的数向右移动操作符右边指定的位数，其中**正数高位补0，负数高位补1**。无符号右移操作符是Java独有的一种，**即不管正数还是负数，右移之后高位都补0**。关于移位操作有两点说明，一是高位指的是左边的位，二是对于int类型，最大长度为32位，对于long类型最大长度为64位。移位示例如下：

```java
package com.chenxyt.java.practice;
public class OperationTest{
	public static void main(String[] args) {
		int a = 6297;
		int b = -6297;
		System.out.println(Integer.toBinaryString(a));
		System.out.println(Integer.toBinaryString(b));
		System.out.println(Integer.toBinaryString(a>>5));
		System.out.println(Integer.toBinaryString(a<<5));
		System.out.println(Integer.toBinaryString(a>>>5));
		System.out.println(Integer.toBinaryString(b>>5));
		System.out.println(Integer.toBinaryString(b<<5));
		System.out.println(Integer.toBinaryString(b>>>5));
	}
}
```

&ensp;&ensp;&ensp;&ensp;我们以int类型6297为例，分别对正6297和负6297做三种位移操作，结果如下：

![png7]([Java编程思想]四：操作符/png7.png)

满足前边所说，左移低位补0，右移正数高位补0，低位补1，因为高位0没有实际意义，所以没有写出。同时，因为数据类型为int类型，所以最大长度为32位，超出的部分被截断了。

&ensp;&ensp;&ensp;&ensp;如果对char、short、byte类型的数值进行移位处理，那么他们会被先转成int类型，并且得到的结果也只是int类型。只有数值右端对的低5位才会有用，这样可以防止移位得到超过int类型的最大位数。比如说5<<34，因为最多能移32位，所以要把34转成二进制，然后取低5位，34转换成2进制是100010，取低5位就是00010，也就是5<<34实际上就是5<<2=10100=20。同样的对于long类型，long的最大长度为64位，所以数值的低6位有效。移位运算符同时也支持“<<=”等运算操作。

# 十二、三元操作符IF-ELSE

&ensp;&ensp;&ensp;&ensp;三元操作符也称条件操作符，它有三个操作符。表达形式如下：
boolean-exp？value1：value2；
boolean-exp是一个布尔表达值，如果值为真，则返回value1的值，如果值为假，则返回value2的值。如：

```java
package com.chenxyt.java.practice;
public class OperationTest{
	public static void main(String[] args) {
		int a = 5;
		int b = a>10?a*100:a*10;
		System.out.println("b===="+ b);
	}
}
```

运行结果：

![png8]([Java编程思想]四：操作符/png8.png)

因为a<10，所以布尔表达式的值为假，所以取value2也就是a*10的值。

# 十三、字符串操作符+和+=

&ensp;&ensp;&ensp;&ensp;Java中可以使用+和+=连接String类型，并且当操作符左边你的数为String时，Java会试图将操作符右边的类型转换成String。

# 十四、使用操作符常犯的错误

&ensp;&ensp;&ensp;&ensp;注意“=”和“==”的区别，以及逻辑运算符如（&&）和位运算符（&）的区别和前自增“++i”和后自增“i++”的区别。注意“==”和“equals（）”的使用，注意别名现象。

# 十五、类型转换操作符

&ensp;&ensp;&ensp;&ensp;Java中有跟其它语言相同的转换方式，（类型）值形式，也可以使用基本类型的包装器的转换方法进行转换。

# 十六、Java没有size of

&ensp;&ensp;&ensp;&ensp;C语言中使用size of来获取程序占用的内存字节大小，目的是确定平台的移植操作，Java中没有这个方法，也就是不需要获取这个值，因为Java在不同的平台下基本数据类型具有相同的大小，可以便捷的移植。

# 十七、总结

&ensp;&ensp;&ensp;&ensp;操作符在各个语言中基本通用，熟练掌握自增自减、逻辑运算、位运算以及移位操作即可，理解“==”和“equals()”的原理，理解别名现象。