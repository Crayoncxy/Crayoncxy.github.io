---
title: 【Java编程思想】十二：持有对象
date: 2018-11-15 15:45:02
tags:
- Java 
- Think in Java
categories: 学习笔记
---

&ensp;&ensp;&ensp;&ensp;在前面的学习过程中，我们使用的都是固定数量的且生命周期已知的对象。而在一些情况中，我们可能需要不确定数量不确切类型的对象，这种创建一个单一的对象显然是不行的了。Java提供了多种支持，比如数组，数组可以保存一组基本数据类型。但是数组的大小是固定的，在更特殊的编程条件下，固定长度显然是不友好的，所以Java类库提供了一套相当完整的容器类来解决这个问题。我们也称作是集合类。本章优先学习常用的集合以及用法，后续将会更加深入的讨论其它的集合。

# 一、泛型和类型安全的容器

&ensp;&ensp;&ensp;&ensp;在JavaSE5泛型的概念出来之前，容器的一个主要问题就是编译器允许你在容器中插入任意类型的对象，这在一些情况下显然是不合理也是不可靠的。考虑这样一个情况，有一个存储Apple对象的容器，我们使用最基本最可靠的ArrayList，ArrayList你可以看成是一个可以自动扩充的数组。ArrayList需要使用索引就像数组下标一样，但是不需要使用方括号，它使用add（）插入对象，使用get（）获取对象，使用size（）获取长度。

&ensp;&ensp;&ensp;&ensp;在本例中Orange对象也会存在Apple容器中，正常情况下编译器会报出警告信息，此处使用@SuppressWarnings注解忽略没有类型检查的警告信息

```java
package com.chenxyt.java.practice;
 
import java.util.ArrayList;
 
class Apple{
	private static long counter;
	private final long id = counter++;
	public long getid(){
		return id;
	}
}
class Orange{
	
}
public class AppleAndOrangesWithoutGeneric {
	@SuppressWarnings("unchecked")
	public static void main(String[] args) {
		ArrayList al = new ArrayList();
		for(int i=0;i<3;i++){
			al.add(new Apple());
			al.add(new Orange());
		}
		for(int i=0;i<al.size();i++){
			Apple apple = (Apple) al.get(i);
			System.out.println(apple.getid());
		}
	}
}
```

&ensp;&ensp;&ensp;&ensp;在这里我们实际上存入ArrayList是Object对象，因为所有的类都继承自Object类，所以这里实际上不光可以添加Orange对象还可以添加任意类型的对象。我们在get（）获取数据的时候，我们以为获取的是一个Apple对象，实际上获取的是一个Obejct的引用，然后强制转换成我们需要的对象。在这里我们强制的将从ArrayList中取出的Object引用强制转换成Apple类型，那么当遇到这个对象类型实际上是Orange类型时，就会发生类型转换错误。所以说这种创建形式非常不安全。报错如下：

![png1]([Java编程思想]十二：持有对象/png1.png)

&ensp;&ensp;&ensp;&ensp;JavaSE5之后引入了泛型的概念，这个概念应用在这种不确定类型的对象集合中最好不过。例如我们如果想创建一个Apple类型的容器，那么就可以使用ArrayList<Apple> 其中<>括起来的是类型参数（可以有多个），它指定了这个容器可以保存的数据类型，在进入容器之前就限制了对象的类型，所以在取出数据的时候就不会发生类型转换错误了。相对来说是更加安全的。

```java
package com.chenxyt.java.practice;
 
import java.util.ArrayList;
 
class Apple{
	private static long counter;
	private final long id = counter++;
	public long getid(){
		return id;
	}
}
class Orange{
	
}
public class AppleAndOrangesWithoutGeneric {
	public static void main(String[] args) {
		ArrayList<Apple> al = new ArrayList<Apple>();
		for(int i=0;i<3;i++){
			al.add(new Apple());
		//	al.add(new Orange());
		}
		for(Apple apple :al){
			System.out.println(apple.getid());
		}
	}
}
```

&ensp;&ensp;&ensp;&ensp;现在你可以阻止将Orange对象加入到Apple容器中了，并且我们可以使用for-each循环获取容器中的内容。实际上当我们固定了一种泛型数据参数，正如前面几章说到的，继承的时候基类收发的消息，子类也可以收发，所以泛型同样支持向上转型，我们可以传递其导出类参数。

# 二、基本概念

&ensp;&ensp;&ensp;&ensp;Java容器的用途是“保存对象”，并将其划分为两个不同的概念。

**&ensp;&ensp;&ensp;&ensp;Collection：**一个独立的元素序列，这些元素都服从一条或多条规则。List必须按照插入的顺序保存对象，而Set不能有重复的元素。Queue按照排队的规则来确定对象的产生顺序。

**&ensp;&ensp;&ensp;&ensp;Map：**一组成对的“键值对”对象，允许你使用键来查找值。ArrayList允许你使用数组查找值，所以某种意义上讲它是将对象与数字进行了绑定。

&ensp;&ensp;&ensp;&ensp;尽管并非总是这样，但是理想情况下我们都是与这些接口打交道。比如，我们可以像下边这样创建一个List:

```java
List<Apple> apples = new ArrayList<Apple>();
```

&ensp;&ensp;&ensp;&ensp;这里ArrayList已经被向上转型为List，使用接口的目的是如何修改它的实现，这里实现由ArrayList完成。这种方式并非永远有效，会带来一个其它的问题，ArrayList类可能新添加了其它的方法，因此当它向上转型为List时，可能会是不完善的。因此如果我们要使用这类方法，那就需要使用更加确切的类型。

# 三、添加一组元素

&ensp;&ensp;&ensp;&ensp;java.util包中Arrays和Collections都有很多实用的方法，可以在一个Collection中添加一组元素。 
Arrays.asList()方法接受一个数组或者一组用逗号分隔的数据元素列表，并将其转化成一个List对象，但是实际底层是Array数组，因此该方法转换之后的对象不支持变更长度。
&ensp;&ensp;&ensp;&ensp;Collection.addAll()方法接受一个Collection对象，以及一个数组或者是一组用逗号分隔的元素列表，将后者添加到前者对象中。
&ensp;&ensp;&ensp;&ensp;下边的例子展示了上边两种方法，以及传统的Collection.addAll()方法：

```java
package com.chenxyt.java.practice;
 
import java.util.ArrayList;
import java.util.Arrays;
import java.util.Collection;
import java.util.Collections;
import java.util.List;
 
public class AddingGroups {
	public static void main(String[] args) {
		Collection<Integer> collection = new ArrayList<Integer>(Arrays.asList(1,2,3,4,5));
		Integer[] moreInts = {6,7,8,9,10};
		collection.addAll(Arrays.asList(moreInts));
		Collections.addAll(collection,11,12,13,14);
		Collections.addAll(collection,moreInts);
		List<Integer> list = Arrays.asList(15,16,17,18);
		list.set(1,29);
		//run error 数组不支持变更长度
		list.add(21);
	}
}
```

&ensp;&ensp;&ensp;&ensp;Collection的构造器可以传递另一个Collection用来初始化，但是这种方式不如定义个空的Collection然后使用addAll的形式添加数据灵活

# 四、容器的打印

&ensp;&ensp;&ensp;&ensp;基本类型的容器都带有toString（）的方法，以便打印。如下示例介绍了几种基本类型的容器的打印。

```java
package com.chenxyt.java.practice;
import java.util.ArrayList;
import java.util.Collection;
import java.util.HashMap;
import java.util.HashSet;
import java.util.LinkedHashMap;
import java.util.LinkedHashSet;
import java.util.LinkedList;
import java.util.Map;
import java.util.TreeMap;
import java.util.TreeSet;
public class PrintingContainers {
	static Collection fill(Collection<String> collection){
		collection.add("one");
		collection.add("two");
		collection.add("three");
		collection.add("four");
		collection.add("five");
		collection.add("five");
		return collection;
	}
	static Map fill(Map<String,String> map){
		map.put("one","ONE");
		map.put("two","TWO");
		map.put("three","THREE");
		map.put("four","FOUR");
		map.put("five","FIVE");
		map.put("five","FIVE");
		return map;
	}
	public static void main(String[] args) {
		System.out.println("ArrayList===" + fill(new ArrayList<String>()));
		System.out.println("LinkedList===" + fill(new LinkedList<String>()));
		System.out.println("HashSet===" + fill(new HashSet<String>()));
		System.out.println("TreeSet===" + fill(new TreeSet<String>()));
		System.out.println("LinkedHashSet===" + fill(new LinkedHashSet<String>()));
		
		System.out.println("HashMap===" + fill(new HashMap<String,String>()));
		System.out.println("TreeMap===" + fill(new TreeMap<String,String>()));
		System.out.println("LinkedHashMap===" + fill(new LinkedHashMap<String,String>()));
	}
}
```

运行结果：

![png2]([Java编程思想]十二：持有对象/png2.png)

&ensp;&ensp;&ensp;&ensp;这里展示了Java容器中的两种主要的类型：Collection和Map，Collection又包括List和Set他们每个位置只能保存一个数据。而Map保存数据的形式则是使用键值对“key-value”的形式。List：以特定的顺序保存数据，Set：集合中的元素都不能重复。Collection还包括另外一种Queue，它要求元素只能从集合的一端进入，从另一端取出。
&ensp;&ensp;&ensp;&ensp;从本例的输出结果可以看出，默认的容器带有的toString方法，可以很好的将容器中的数据展示出来。Collection使用[]括起来，Map使用{}括起来，键值对使用=连接。
&ensp;&ensp;&ensp;&ensp;接着分析打印输出，ArrayList和LinkedList都是List类型，它们能够按照元素的填入顺序进行打印。区别在于执行某些操作时候的性能不同，而且LinkedList的功能要多于ArrayList，这在后文介绍。
&ensp;&ensp;&ensp;&ensp;HashSet、TreeSet、LinkedHashSet都是Set类型，每个相同的元素只保存一个，HashSet使用了相当复杂的存储结构，后文会介绍，因此HashSet的存储顺序没有实际意义。TreeSet按元素比较结果的升序排序，LinkedHashSet与List相同，按照元素的插入顺序进行了排序。
&ensp;&ensp;&ensp;&ensp;Map也可以称作是关联数组，可以使用key查找对应的value，同时可以不用关心它们的大小。Map会自动的扩容。同时也不需要关心打印的顺序。

# 五、List

&ensp;&ensp;&ensp;&ensp;List将元素按照插入的顺序排列起来，它在Collection的接口中增加了新的功能，使得可以在List的中间插入和删除元素。List有两种类型如上一节介绍:
&ensp;&ensp;&ensp;&ensp;1.ArrayList 优势在于随机访问的速度很快，但是在List中间插入和删除元素比较慢。
&ensp;&ensp;&ensp;&ensp;2.LinkedList 与上基本相反，它的特性集较ArrayList更大。
&ensp;&ensp;&ensp;&ensp;下面的一个例子展示List的一些特性，首先有个枚举类型，它有几个常量，然后是一个向List中增加数据的方法，这个方法我们使用的是ArrayList，注意前边说到，ArrayList增删操作性能消耗很大，如果我们的程序设计中出现了大量的ArrayList插入操作，那么可能会导致性能的降低

```java
package com.chenxyt.java.practice;
import java.util.ArrayList;
import java.util.List;
import java.util.Random;
enum Pet{
	Pet,Rat,Manx,Mutt,Pug,Cymric,Humaster
}
public class Pets {
	public static List<Pet> arrayList(int n){
		List<Pet> pets = new ArrayList<Pet>();
		for(int i=0;i<n;i++){
			Random rand = new Random();
			int j = rand.nextInt(n);
			switch(j){
			case 0:
				pets.add(Pet.Pet);
				break;
			case 1:
				pets.add(Pet.Rat);
				break;
			case 2:
				pets.add(Pet.Manx);
				break;
			case 3:
				pets.add(Pet.Mutt);
				break;
			case 4:
				pets.add(Pet.Pug);
				break;
			case 5:
				pets.add(Pet.Cymric);
				break;
			case 6:
				pets.add(Pet.Humaster);
				break;
				default:
					break;
			}
		}
		return pets;
	}
}
```

&ensp;&ensp;&ensp;&ensp;然后是一个List的操作类，相关操作的注释已经标注：

```java
package com.chenxyt.java.practice;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.Collections;
import java.util.List;
import java.util.Random;
public class ListFeature {
	public static void main(String[] args) {
		Random rand = new Random(47);
		List<Pet> pets = Pets.arrayList(7);
		//基本的List展示
		System.out.println("1:" + pets);
		pets.add(Pet.Humaster);
		//List可以自动扩容 向其中添加的元素会被追加到最后
		System.out.println("2:" + pets);
		//a.contains(b)方法判断 集合a中是否包含b 返回的是boolean类型 
		System.out.println("3:" + pets.contains(Pet.Humaster));
		//remove 移除指定的元素
		pets.remove(Pet.Humaster);
		Pet p = pets.get(2);
		//get（2）获取指定位置上的元素 下标从0开始  indexOf(p) 返回指定元素的下标
		System.out.println("4:" + p + " " + pets.indexOf(p));
		Pet cymric = Pet.Cymric;
		//indexOf(p) 返回指定元素的下标 如果没有则返回-1 存在多个则返回第一个
		System.out.println("5:" + pets.indexOf(cymric));
		//remove（p） 移除指定元素 返回boolean类型 不存在为false 存在多个则移除第一个
		System.out.println("6:" + pets.remove(cymric));
		System.out.println("7:" + pets.remove(p));
		System.out.println("8:" + pets);
		//在下标为3的位置插入元素，后边的元素顺序后移
		pets.add(3,Pet.Mutt);
		System.out.println("9:" + pets);
		//截断 下标为1到4的元素 包括1但是不包括4
		List<Pet> sub = pets.subList(1,4);
		System.out.println("subList:" + sub);
		//a.containsAll(b) a集合是否包含b集合中的全部元素 返回boolean类型
		System.out.println("10:" + pets.containsAll(sub));
		//排序
		Collections.sort(sub);
		System.out.println("sorted subList:" + sub);
		System.out.println("11:" + pets.containsAll(sub));
		//随机排序
		Collections.shuffle(sub,rand);
		System.out.println("suffle subList:" + sub);
		System.out.println("12:" + pets.containsAll(sub));
		List<Pet> copy = new ArrayList<Pet>(pets);
		sub = Arrays.asList(pets.get(1),pets.get(4));
		System.out.println("sub:" + sub);
		//a.retainAll(b) a中保留所有与b的交集部分
		copy.retainAll(sub);
		System.out.println("13:" + copy);
		copy = new ArrayList<Pet>(pets);
		copy.remove(2);
		System.out.println("14:" + copy);
		//移除所有的元素
		copy.removeAll(sub);
		System.out.println("15:" + copy);
		//将下标为1的元素更换为指定元素
		copy.set(1,Pet.Pug);
		System.out.println("16:" + copy);
		copy.addAll(2,sub);
		System.out.println("17:" + copy);
		//判断是否为空 返回boolean类型
		System.out.println("18:" + pets.isEmpty());
		//清空数据
		pets.clear();
		System.out.println("19:" + pets);
		System.out.println("20:" + pets.isEmpty());
		pets.addAll(Pets.arrayList(4));
		System.out.println("21:" + pets);
		//将List转换成数组
		Object[] o = pets.toArray();
		System.out.println("22:" + o[3]);
	}
}
```

运行结果：

![png3]([Java编程思想]十二：持有对象/png3.png)

&ensp;&ensp;&ensp;&ensp;因为数据的插入是随机的，所以后边对应的操作也是随机结果。相关操作已经写在注释中，这里不再阐述。

# 六、迭代器

&ensp;&ensp;&ensp;&ensp;迭代器（也是一种设计模式），是一种应用在容器之上的设计。它是一个对象，它的作用是遍历并选择序列中的对象，而不需要确定该序列的底层机构。也就是说，我们使用迭代器的目的是解决了不同容器之间的互通性问题。因此迭代器通常被称作是“轻量级对象”，创建它的代价较小。因此通常有些奇怪的限制。例如：Java中的迭代器只能单向移动，并且它通常：
&ensp;&ensp;&ensp;&ensp;1、使用方法iterator（）要求容器返回一个Iterator，Iterator将准备好返回容器的第一个元素。
&ensp;&ensp;&ensp;&ensp;2、使用next（）方法获得容器中的下一个元素。
&ensp;&ensp;&ensp;&ensp;3、使用hasNext（）方法检查容器中是否还有下一个元素。
&ensp;&ensp;&ensp;&ensp;4、使用remove（）方法将迭代器中新近返回的元素删除。
&ensp;&ensp;&ensp;&ensp;我们继续使用上节的Pets类进行Iterator的测试

```java
package com.chenxyt.java.practice;
import java.util.Iterator;
import java.util.List;
public class SimpleIterator {
	public static void main(String[] args) {
		List<Pet> pets = Pets.arrayList(7);
		Iterator<Pet> it = pets.iterator();
		System.out.println("1:" + pets);
		System.out.println("2:" + it);
		while(it.hasNext()){
			Pet p = it.next();
			System.out.println("--->" + p);
		}
		it = pets.iterator();
		for(int i = 0;i<7;i++){
			it.next();
			it.remove();
			System.out.println(pets);
		}
	}
}
```

运行结果：

![png4]([Java编程思想]十二：持有对象/png4.png)

&ensp;&ensp;&ensp;&ensp;需要注意的是，第二行我们打印Iterator时并没有像打印其它对象时把数据打印出来，而是只打印了这个迭代器底层容器的类型ArrayList。还有就是迭代器对象第一次.next之后获取的是第一个元素，remove方法移除的是next方法获取到的元素，所以remove之前必须要调用next方法。在同一方法中使用Iterator时，要注意前面使用过之后，Iterator的位置会发生变化。
&ensp;&ensp;&ensp;&ensp;现在我们换用其它类型的容器来测试迭代器的作用，这里有个display（）方法，它不考虑任何容器的类型来进行打印：

```java
package com.chenxyt.java.practice;
import java.util.ArrayList;
import java.util.HashSet;
import java.util.Iterator;
import java.util.LinkedHashSet;
import java.util.LinkedList;
import java.util.TreeSet;
public class SimpleIterator {
	public static void display(Iterator<Pet> it){
		System.out.println("===" +it.getClass().getName()+ "===");
		while(it.hasNext()){
			Pet p = it.next();
			System.out.println(p);
		}
	}
	public static void main(String[] args) {
		ArrayList<Pet> pets = (ArrayList<Pet>) Pets.arrayList(7);
		LinkedList<Pet> lkp = new LinkedList<Pet>(pets);
		HashSet<Pet> hs = new HashSet<Pet>(pets);
		TreeSet<Pet> ts = new TreeSet<Pet>(pets);
		LinkedHashSet<Pet> lhs = new LinkedHashSet<Pet>(pets);
		display(pets.iterator());
		display(lkp.iterator());
		display(hs.iterator());
		display(ts.iterator());
		display(lhs.iterator());
	}
}
```

运行结果：

![png5]([Java编程思想]十二：持有对象/png5.png)

&ensp;&ensp;&ensp;&ensp;还有一种比Iterator更加强大的迭代器ListIterator，这个迭代器功能更全，可以向前或者向后移动，也可以使用set（）方法替换它最后访问的元素（使用方式如Iterator的remove方法，要先指定访问元素），同时它还可以使用listIterator（n）方法直接指定到第n个元素的ListIterator。下面的示例展示了ListIterator的功能：

```java
package com.chenxyt.java.practice;
import java.util.List;
import java.util.ListIterator;
public class SimpleListIterator {
	public static void main(String[] args) {
		List<Pet> pets = Pets.arrayList(7);
		ListIterator<Pet> lit = pets.listIterator();
		System.out.println(pets);
		System.out.println(lit);
		System.out.println("=====Next=====");
		while(lit.hasNext()){
			int index = lit.nextIndex();
			Pet p = lit.next();
			System.out.println("Index:" + index + "," + "Pet:" + p );
		}
		System.out.println("=====Previous=====");
		lit = pets.listIterator(7);
		while(lit.hasPrevious()){
			int index = lit.previousIndex();
			Pet p = lit.previous();
			System.out.println("Index:" + index + "," + "Pet:" + p );
		}
		System.out.println("=====Update Set=====");
		lit = pets.listIterator();
		while(lit.hasNext()){
			Pet p = lit.next();
			lit.set(Pet.Humaster);
		}
		System.out.println(pets);
	}
}
```

运行结果：

![png6]([Java编程思想]十二：持有对象/png6.png)

&ensp;&ensp;&ensp;&ensp;如上是ListIterator的一些特性，相比Iterator，功能更加全面了一些。需要注意的是，nextIndex和previous不会发生移位操作。

# 七、LinkedList

&ensp;&ensp;&ensp;&ensp;LinkedList与ArrayList相同都是实现了List接口，但是它在增删的时候效率较高，在随机访问的时候效率略低。LinkedList还增加了其作为栈、队列、双端队列的操作方法。这些方法有的只是名字有差异，或者是返回值有差异。

```java
package com.chenxyt.java.practice;
import java.util.LinkedList;
public class LinkedListFeatures {
	public static void main(String[] args) {
		LinkedList<Pet> pets = new LinkedList<Pet>(Pets.arrayList(7));
		System.out.println(pets);
		//以下两个方法都返回容器的第一个元素 在容器为空的时候抛异常
		System.out.println("pets.getFirst()---》" + pets.getFirst());
		System.out.println("pets.elements()---》" + pets.element());
		//与上两个方法相同 区别在于为空时返回null
		System.out.println("pets.peek()---》" + pets.peek());
		//以下两个方法移除列表的第一个元素 在容器为空的时候抛出异常
		System.out.println("pets.remove()--->" + pets.remove());
		System.out.println("pets.removeFirst()--->" + pets.removeFirst());
		//与上两个方法相同 区别在于容器为空时返回null
		System.out.println("pets.pool()--->" + pets.poll());
		System.out.println(pets);
		//在容器第一个位置加入新的元素 其它元素依次后移
		pets.addFirst(Pet.Manx);
		System.out.println("After addFirst()" + pets);
		//以下两个方法在容器尾部插入新元素
		pets.add(Pet.Humaster);
		System.out.println("After add()" + pets);
		pets.addLast(Pet.Pug);
		System.out.println("After addLast()" + pets);
		//offer 是针对queue 在尾部插入数据 add是针对list 在尾部插入数据
		pets.offer(Pet.Rat);
		System.out.println("After offer()" + pets);
		//移除最后一个并返回该元素
		System.out.println("pets.removeLast()" + pets.removeLast());
		System.out.println("After removeLast()" + pets);
	}
}
```

运行结果：

![png7]([Java编程思想]十二：持有对象/png7.png)

# 八、Stack

&ensp;&ensp;&ensp;&ensp;“栈”通常是指“后进先出的容器”，比如装羽毛球的桶，最后放进去的羽毛球可以第一个被拿出来，因为另一端也就是栈底是封闭的。LinkedList具有能够直接实现栈的所有功能的方法，因此可以直接将LinkedList作为栈使用。这里我们使用一个真正的Stack，内部使用LinkedList来实现它。

```java
package com.chenxyt.java.practice;
import java.util.LinkedList;
public class Stack<T> {
	private LinkedList<T> storage = new LinkedList<T>();
	public void push(T v){
		storage.addFirst(v);
	}
	public T peek(){
		return storage.getFirst();
	}
	public T pop(){
		return storage.removeFirst();
	}
	public boolean empty(){
		return storage.isEmpty();
	}
	public String toString(){
		 return storage.toString();
	}
}
```

&ensp;&ensp;&ensp;&ensp;这里使用<T>泛型来告诉编译器这个Stack是持有参数化类型T的容器，这个Stack是使用LinkedList实现的，而LinkedList也被告知是使用了T类型的对象。peek（）方法返回栈顶元素，但是并不是移除。而pop（）方法这里是弹出栈顶元素，也就是移除了栈顶元素。如果我们只是需要栈的行为，而不需要其它无关的行为方法，那么这里使用继承就显然不合适了。后边会讨论在Java1.0中，java.util.Stack这个类的设计。

&ensp;&ensp;&ensp;&ensp;下面演示了我们如何使用上边这个新的Stack类：

```java
package com.chenxyt.java.practice;
public class StackTest {
	public static void main(String[] args) {
		Stack<String> stack = new Stack<String>();
		for(String s : "My Dog has fleas".split(" ")){
			stack.push(s);
		}
		while(!stack.empty()){
			System.out.print(stack.pop() + " ");
		}
	}
}
```

&ensp;&ensp;&ensp;&ensp;这里使用了push方法将字符串“My Dog has fleas”用空格分开的单词压入栈中，然后使用pop方法弹出栈顶元素。因为pop方法调用之后会移除栈顶元素，所以会依次弹出栈中的所有元素。在这里我们使用了自己定义的Stack，如果我们导入了java.util.Stack类的话，那么我们这样使用可能会产生命名冲突的现象，解决方法是我们在实例化的时候使用完整的类名，或者修改我们自己定义的Stack类名。

# 九、Set

&ensp;&ensp;&ensp;&ensp;Set不保存重复的元素（至于如何判断元素是否重复，则较为复杂，稍后便会看到），Set最常见的使用是判断对象的归属性，即判断某个对象是否在Set中。正因为如此，Set的查询效率就显得尤为重要了，所以通常会使用HashSet来实现所需要的功能，它对查询专门做了优化。

&ensp;&ensp;&ensp;&ensp;Set具有与Collection完全一样的接口，我们可以说Set就是Collection，这是表现了不同的功能，这是Java中继承与多态思想的展现。Set中更加复杂的问题后边17章会介绍。

&ensp;&ensp;&ensp;&ensp;下面示例使用了HashSet存放Integer对象：

```java
package com.chenxyt.java.practice;
import java.util.HashSet;
import java.util.Random;
import java.util.Set;
public class SetofInteger {
	public static void main(String[] args) {
		Random random = new Random(47);
		Set<Integer> set = new HashSet<Integer>();
		for(int i=0;i<10000;i++){
			set.add(random.nextInt(30));
		}
		System.out.println(set);
	}
}
```

运行结果：

![png8]([Java编程思想]十二：持有对象/png8.png)

&ensp;&ensp;&ensp;&ensp;set中插入了0-29的随机数，插入了10000次，但是从打印结果可以看出，只保证了每个数据只出现了一次。而且似乎插入的顺序并没有规律可寻。这是因为出于查找速度的考虑，HashSet使用了散列，将在17章中介绍。HashSet使用散列存储，TreeSet使用红黑树存储，LinkedHahSet因为查询速度的原因也使用了散列，但是它看起来像是使用了链表来进行存储，因为它保证了元素的插入顺序。

&ensp;&ensp;&ensp;&ensp;注意：插入顺序与元素的顺序的区别，插入顺序是我们在容器中添加元素时的顺序，而元素的顺序说的是按照某种规则比如从小到大或者从大到小的顺序，也就是元素的结果。当然，上边的示例如果我们相对元素的结果进行排序，那么可以使用TreeSet来替换，因为LinkedHashSet只是保证了元素的插入顺序。

&ensp;&ensp;&ensp;&ensp;Set 常用的方法是使用contains（）判断是否存在某元素，当然还有一些其它的顾名思义的方法：

```java
package com.chenxyt.java.practice;
import java.util.Collections;
import java.util.HashSet;
import java.util.Set;
public class SetOperations {
	public static void main(String[] args) {
		Set<String> set1 = new HashSet<String>();
		Collections.addAll(set1,"A B C D E F G H I J K L".split(" "));
		set1.add("M");
		System.out.println("H:" +  set1.contains("H"));
		System.out.println("N:" +  set1.contains("N"));
		Set<String> set2 = new HashSet<String>();
		Collections.addAll(set2,"H I J K L".split(" "));
		System.out.println("set2 in set1:" + set1.containsAll(set2));
		set1.remove("H");
		System.out.println("set1: " + set1);
		System.out.println("set2 in set1:" + set1.containsAll(set2));
		set1.removeAll(set2);
		System.out.println("set2 removed from set1:" + set1);
		Collections.addAll(set1,"X Y Z".split(" "));
		System.out.println("XYZ added to set1:" + set1);
	}
}
```

运行结果：

![png9]([Java编程思想]十二：持有对象/png9.png)

# 十、Map

&ensp;&ensp;&ensp;&ensp;Map可以将一个对象映射到另一个对象上组建一种一对一的键值关系。比如可以设计这样一个程序来验证Java中Random的随机性，理想情况下，Random对每个随机数产生的概率是相同的。我们测试这一理论，于是定义一个Map，键来表示随机出现的数字，值来表示该数字出现的次数。

```java
package com.chenxyt.java.practice;
import java.util.HashMap;
import java.util.Map;
import java.util.Random;
public class RandomTest {	
	public static void main(String[] args) {
		Random random = new Random(47);
		Map<Integer,Integer> map = new HashMap<Integer,Integer>();
		for(int i =0;i<1000000;i++){
			int x = random.nextInt(20);
			Integer freq = map.get(x);
			map.put(x,freq == null?1:freq+1);
		}
		System.out.println(map);
	}
}
```

运行结果：

![png10]([Java编程思想]十二：持有对象/png10.png)

![png11]([Java编程思想]十二：持有对象/png11.png)

&ensp;&ensp;&ensp;&ensp;20个数字随机1000000次，每个数字出现的概率为1/50000，由运行结果可以看出，基本符合这一理想情况下的推论。上述main（）方法中所用的map.get（）方法，获取指定键对应的值，如果该键不存在则返回null。Map同样具有判断键值是否存在的方法。

```java
package com.chenxyt.java.practice;
import java.util.HashMap;
import java.util.Map;
public class PetMap {
	public static void main(String[] args) {
		Map<String,Pet> petMap = new HashMap<String,Pet>();
		petMap.put("MyCat",Pet.Cymric);
		petMap.put("MyDog",Pet.Humaster);
		petMap.put("MyPig",Pet.Mutt);
		System.out.println(petMap);
		Pet p = petMap.get("MyDog");
		System.out.println(p);
		System.out.println(petMap.containsKey("MyDog"));
		System.out.println(petMap.containsValue(Pet.Cymric));
	}
}
```

运行结果：

![png12]([Java编程思想]十二：持有对象/png12.png)

&ensp;&ensp;&ensp;&ensp;Map与数组或者其它的Collection一样，可以扩展到多维角度，即key是String类型，value可以重新定义为一个容器。比如上边的示例改造为一个动物有多个名字。那么你需要的就是一个Map<String,List<Pet>>:

```
package com.chenxyt.java.practice;
 
import java.util.Arrays;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
public class MapOfList {
	public static void main(String[] args) {
		 Map<String,List<Pet>> petList = new HashMap<String,List<Pet>>();
		 petList.put("MyDog",Arrays.asList(Pet.Cymric,Pet.Humaster,Pet.Mutt));
		 System.out.println(petList);
		 System.out.println("key:" + petList.keySet());
		 System.out.println("value:" + petList.values());
		 for(String s:petList.keySet()){
			 System.out.print(s + ":");
			 for(Pet p:petList.get(s)){
				 System.out.print(p + " ");
			 }
		 }
	}
}
```

运行结果：

![png13]([Java编程思想]十二：持有对象/png13.png)

&ensp;&ensp;&ensp;&ensp;上述代码中还展示了.keySet（）方法和.values（）方法，分别返回Map中所有的键和所有的值。

# 十一、Queue

&ensp;&ensp;&ensp;&ensp;队列是一个典型的先进先出的容器，就像是一个两端打开的管子，从一端放进去的物品，从另一端取出，并且最先取出的是最先放进去的物品。因此队列的取出顺序往往是与插入顺序相同的。正因为队列这种顺序特性，它常常被作为一种可靠的将对象从程序的某个区域发送到另一个区域的途径。LinkedList实现了Queue的接口，因此LinkedList可以当做是Queue的一种实现，这是面向对象编程中继承与多态思想的体现。将LinkedList向上转型为Queue，下面的示例将展示Queue接口中与Queue相关的方法：

```java
package com.chenxyt.java.practice;
import java.util.LinkedList;
import java.util.Queue;
public class QueueDemo {
	public static void printQ(Queue queue){
		//不移除的情况下返回队列头部元素  element（）方法相同  peek为空返回null  element为空抛出异常
		while(queue.peek()!=null){
			//remove（）移除头元素 为空抛出异常 poll 为空返回null
			System.out.println(queue.remove() + " ");
		}
	}
	public static void main(String[] args) {
		Queue<Integer> queue = new LinkedList<Integer>();
		for(int i = 0;i<5;i++){
			//元素插入队尾
			queue.offer(i);
		}
		printQ(queue);
		Queue<Character> qc = new LinkedList<Character>();
		for(char c:"QUEUEDEMO".toCharArray()){
			qc.offer(c);
		}
		printQ(qc);
	}
}
```

运行结果：

![png14]([Java编程思想]十二：持有对象/png14.png)

&ensp;&ensp;&ensp;&ensp;offer方法是队列的相关方法之一，它在队列允许的情况下，将元素插入到队列的尾部，或者返回false，peek和element方法都是在不删除的情况下返回队列的第一个元素，peek在队列为空时返回null，element在队列为空抛NoSuchElementException，poll和remove删除当前队列的第一个元素，如果为空poll返回null，remove抛NoSuchElementException异常。

&ensp;&ensp;&ensp;&ensp;先进先出描述了最典型的队列规则，在某些情况下，队列还需要弹出当前最需要的元素，这种队列称作是优先级队列。优先级队列每个元素都具有一个执行的优先级，也就是这个元素何时弹出与何时插入没有必然的联系。PriorityQueue添加到Java SE5中，是为了这种优先级形式自动实现。当我们在PriorityQueue上调用了offer（）方法来插入一个对象时，这个对象在队列中就会被重新排序。默认的排序是对象在队列中的自然顺序。但是你可以通过提供自己的Comparator来修改这个顺序。PriorityQueue可以确保你在调用peek、poll、remove等方法时获取的是当前队列中优先级最高的元素。

```java
package com.chenxyt.java.practice;
import java.util.Arrays;
import java.util.Collections;
import java.util.List;
import java.util.PriorityQueue;
import java.util.Queue;
public class QueueDemo {
	public static void printQ(Queue queue){
		//不移除的情况下返回队列头部元素  element（）方法相同  peek为空返回null  element为空抛出异常
		while(queue.peek()!=null){
			//remove（）移除头元素 为空抛出异常 poll 为空返回null
			System.out.println(queue.remove() + " ");
		}
	}
	public static void main(String[] args) {
		List<Integer> ints = Arrays.asList(25,22,20,18,14,9,3,1,5,6,14,18,22,23,25);
		PriorityQueue<Integer> queue = new PriorityQueue<Integer>(ints.size(),Collections.reverseOrder());
		queue.addAll(ints);
		printQ(queue);
	}
}
```

运行结果：

![png15]([Java编程思想]十二：持有对象/png15.png)

&ensp;&ensp;&ensp;&ensp;我们使用了JavaSE5中定义的reverseOrder反序定义了PriorityQueue的优先级。

# 十二、Collection和Iterator

 &ensp;&ensp;&ensp;&ensp;Collection是描述所有序列容器共性的根接口：

```java
package com.chenxyt.java.practice;
import java.util.Collection;
import java.util.HashSet;
import java.util.Iterator;
import java.util.LinkedHashMap;
import java.util.List;
import java.util.Map;
import java.util.Set;
public class InterfaceVsIterator {
	public static void display(Iterator<Pet> it){
		System.out.println("IT-P:");
		while(it.hasNext()){
			Pet p = it.next();
			System.out.print(p + ",");
		}
		System.out.println("");
	}
	public static void display(Collection<Pet> collection){
		System.out.println("CL-P:");
		for(Pet p:collection){
			System.out.print(p + ",");
		}
		System.out.println("");
	}
	public static void main(String[] args) {
		List<Pet> petList = Pets.arrayList(7);
		Set<Pet> petSet = new HashSet<Pet>(petList);
		Map<String,Pet> petMap = new LinkedHashMap<String,Pet>();
		String[] names = ("A,B,C,D,E").split(",");
		for(int i =0;i<names.length;i++){
			petMap.put(names[i],petList.get(i));
		}
		display(petList);
		display(petSet);
		display(petList.iterator());
		display(petSet.iterator());
		System.out.println(petMap);
		System.out.println(petMap.keySet());
		display(petMap.values());
		display(petMap.values().iterator());
	}
}
```

运行结果：

![png16]([Java编程思想]十二：持有对象/png16.png)

&ensp;&ensp;&ensp;&ensp;从运行结果可以看出，使用Collection和Iterator看起来没有什么区别，通常情况下，使用Collection要更方便一些。

# 十三、Foreach与迭代器

&ensp;&ensp;&ensp;&ensp;foreach语法可以应用在数组中，也可以应用在任何Collection对象，之所以能够这样，是因为JavaSE5引入了新的被称为Iterable的接口，该接口包含了一个能够产生Iterator的iterator（）方法，并且Iterable接口被foreach用来在序列中移动。因此如果你创建了任何实现Iterabel的类，都可以将它应用在foreach中。

```java
package com.chenxyt.java.practice;
 
import java.util.Iterator;
 
public class IterableClass implements Iterable<String> {
	protected String[] words = ("And that is how we know the earth").split(" ");
	@Override
	public Iterator<String> iterator() {
		// TODO Auto-generated method stub
		return new Iterator<String>(){
			private int index = 0;
			public boolean hasNext(){
				return index<words.length;
			}
			public String next(){
				return words[index++];
			}
			public void remove(){
				//---
			}
		};
	}
	
	public static void main(String[] args) {
		for(String s :new IterableClass()){
			System.out.print(s + " ");
		}
	}
}
```

运行结果：

![png17]([Java编程思想]十二：持有对象/png17.png)

&ensp;&ensp;&ensp;&ensp;iterator（）方法返回的实现了Iterator<String> 的匿名内部类的实例，该匿名内部类可以遍历数组中的所有单词。在main（）中，你可以看到IteratorClass确实可以用于foreach语句中。

# 十四、总结

&ensp;&ensp;&ensp;&ensp;容器也就是集合，是Java基础中很重要的一个模块，本篇文章了解了基础的容器使用方式。Java提供了多种容器持有对象的方式：
&ensp;&ensp;&ensp;&ensp;1.数组将数字也就是下标与内容关联，查询时不需要进行类型转换，缺点是大小一旦确定，则不能被改变。
&ensp;&ensp;&ensp;&ensp;2.Collection保存单一的元素，Map保存相关联的键值对。
&ensp;&ensp;&ensp;&ensp;3.像数组一样，List也建立数字索引与对象的关联，因此数组和List都是排序好的容器。
&ensp;&ensp;&ensp;&ensp;4.如果要进行大量的随机访问，就要使用ArrayList，如果要进行大量的插入删除操作，就要使用LinkedList。
&ensp;&ensp;&ensp;&ensp;5.各种队列与栈的操作，由LinkedList提供支持。
&ensp;&ensp;&ensp;&ensp;6.Map是一种将对象与对象进行关联的容器，HashMap设计用来快速访问，TreeMap保持键始终处于排序状态，所以没有HashMap快。LinkedHashMap保持元素插入的顺序，但是也通过散列提供快速访问的能力。
&ensp;&ensp;&ensp;&ensp;7.Set不接受重复的数据，HashSet提供最快的查询速度，而TreeSet保持元素处于排序状态。LinkedHashSet以插入顺序保持元素。

&ensp;&ensp;&ensp;&ensp;简单的容器分类：

![png18]([Java编程思想]十二：持有对象/png18.png)