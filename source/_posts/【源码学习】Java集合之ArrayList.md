---
layout: posts
title: 【源码学习】Java集合之ArrayList
date: 2019-03-25 21:35:17
tags:
- 源码
- 集合框架  
categories: 源码学习
typora-copy-images-to: 【源码学习】Java集合之ArrayList
---

# 一、概述

本文不打算将ArrayList全部代码列出来（也着实没有什么必要和精力），打算就常用常见的几个功能进行分析学习（源码系列文章凡是没有特殊说明的均为jdk1.8版本）。主要内容如下：

1. ArrayList的构造器
2. 向ArrayList中添加元素
3. 从ArrayList中获取元素
4. 删除ArrayList中的元素
5. 判断一个元素是否存在于ArrayList中
6. ArrayList中用于遍历的迭代器

如果懒得一步一步的看的话，那么可以直接点击每一个章节中的小结看结论总结部分。

# 二、源码分析

## 2.1ArrayList的构造器

ArrayList有三种构造器，分别如下：

### 2.1.1ArrayList(int initialCapacity)

带有int类型参数的构造器

形如：

```java
public List<String> s = new ArrayList<String>(20);
```

源码：

```java
    public ArrayList(int initialCapacity) {
        //如果初始化大小>0
        if (initialCapacity > 0) {
            //初始化一个指定大小的Object类型数组
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            // ==0则初始化一个空的数组
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            //不能初始化负数
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }
```

这里有几个基本的变量概念：

```java
//可以理解为ArrayList的基本核心数据类型    
transient Object[] elementData; // non-private to simplify nested class access
//空数组
private static final Object[] EMPTY_ELEMENTDATA = {};
```

没错，既然是ArrayList，那么它的核心基本数据类型一定是一个Array，一个Object类型的Array，修饰成transient表示这个属性在用Java默认的序列化方式传输时，不可被序列化。所谓Java默认的序列化方式就是实现Serializable接口的这种形式。

介绍完elementData这个基本的参数概念之后，上边构造器的代码就如同注释所说，初始化一个指定了容量的Object类型的数组，如果容量为0则初始化一个空的数组，为负数则抛出异常。

### 2.1.2ArrayList()

无参数构造器，我一般常用这种形式，因为一般情况下我并不是很确认我需要初始化一个多大容量的构造器。

```java
    /**
     * Constructs an empty list with an initial capacity of ten.
     */
    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }
```

无参构造器就是初始化一个DEFAULTCAPACITY_EMPTY_ELEMENTDATA默认大小的Object数组，那么我们看一下这个DEFAULTCAPACITY_EMPTY_ELEMENTDATA是多大呢？

```java
    /**
     * Shared empty array instance used for default sized empty instances. We
     * distinguish this from EMPTY_ELEMENTDATA to know how much to inflate when
     * first element is added.
     */
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
```

哦豁，仍然是一个空的数组。但是我们观察注释可以发现（上边两段代码的注释），为了与EMPTY_ELEMENTDATA进行区分，这里在添加第一个元素的时候，会扩容为10，也就是初始化了一个大小为10的Object类型数组。具体扩容的事儿后边再说。

### 2.1.3ArrayList(Collection<? extends E> c)

带有Collection集合元素参数的构造器

```java
    public ArrayList(Collection<? extends E> c) {
        //将集合元素转换成数组
        elementData = c.toArray();
        //如果大小不等于0
        if ((size = elementData.length) != 0) {
            // c.toArray might (incorrectly) not return Object[] (see 6260652)
            //如果elementData的类型不为Object数组类型 则重新拷贝
            if (elementData.getClass() != Object[].class)
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            // replace with empty array.
            //==0返回一个空的数组
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }
```

这里有一个size变量，表示的是当前集合中元素的个数，并不是集合的容量。这段带有Collection参数的构造器可以理解为初始化一个拥有一些元素的集合。这里为什么在toArray（）之后又判断了一次elementData是不是Object类型的数组呢？源码中有一行注释：

```java
         // c.toArray might (incorrectly) not return Object[] (see 6260652)
```

译为c.toArray 有可能不会返回一个Object类型的数组，这是jdk之前版本源码的一个bug，所以这里进行一个判断，如果类型变了的话就采用拷贝的方法复制入参数组。

### 2.1.4小结

ArrayList底层的基本数据类型是数组，它提供了三种构造器，一种是无参的（它在添加第一个元素的时候会进行数组扩容），一种是指定数组初始化长度的，一种是指定了初始化内容的。通常情况下我们建议使用第二种指定数组初始化长度的，这样可以节省空间，也可以避免数组扩容降低效率。

## 2.2向ArrayList中添加元素

### 2.2.1boolean add(E e)

向集合中添加一个E类型的元素，添加成功之后返回true

```java
    public boolean add(E e) {
        //判断数组容量是否够存
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        //将元素插入数组下一个空缺位置 然后计数+1 （因为下标从0开始 而计数从1开始）
        elementData[size++] = e;
        return true;
    }
```

这里看上去比较简单了，就是判断容量是否够用，然后添加元素。这里我们需要把ensureCapacityInternal这个方法展开，里边有一些关于ArrayList比较核心的内容。

#### 2.2.1.1ensureCapacityInternal（int）

这个方法用来判断数组容量是否够存，即是否需要扩容

```java
    private void ensureCapacityInternal(int minCapacity) {
        //判断数组是否是通过无参构造函数进行的初始化并且是第一次添加元素
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            //取最大值 根据条件 如果进入到这里 最大值必然为 DEFAULT_CAPACITY
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }
        //判断数组是否需要扩容
        ensureExplicitCapacity(minCapacity);
    }
```

这里有个细节的地方在前文说到，当数组被无参构造函数初始化时，如同DEFAULTCAPACITY_EMPTY_ELEMENTDATA注释所说，在第一次添加元素的时候，数组会被扩容到大小为10的容量。这里的if判断，如果条件成立，则表示当前数组处于刚被无参构造函数初始化完，并没有加入元素。所以方法入参的minCapacity必然等于0+1也就是1，那么Math.max（）方法返回的最大值也就必然是DEFAULT_CAPACITY，那么我们看下这个DEFAULT_CAPACITY值

```java
    /**
     * Default initial capacity.
     */
    private static final int DEFAULT_CAPACITY = 10;

```

好，这回对上了，无参构造函数第一次新增元素的时候被扩容成10个大小。

#### 2.2.1.2ensureExplicitCapacity（int）

这个方法用来判断是否需要进行扩容

```java
    private void ensureExplicitCapacity(int minCapacity) {
        //修改次数+1
        modCount++;

        // overflow-conscious code
        //扩容条件 当添加完新元素之后的大小比数组当前的长度（不是已有元素长度，是初始化长度即包括未填充元素长度，数组已填充长度为size）大，则扩容
        if (minCapacity - elementData.length > 0)
            //扩容
            grow(minCapacity);
    }
```

这里又出现了一个基本的全局变量，这个变量是ArrayList的基类AbstractList的变量，用来记录修改的次数。

```java
//表示ArrayList修改的次数    
protected transient int modCount = 0;
```

方法的逻辑就是先记录一下修改次数，如果入参的数组长度（即添加完新元素之后的最大长度）比数组当前的长度（包括未填充元素的地方，已填充的长度是size）大，那么就进行扩容。

#### 2.2.1.3grow（int）

这个方法就是扩容的方法，可见如果我们能确定ArrayList元素的个数，那么直接初始化会省事的多，因为避免了扩容的过程。

```java
    private void grow(int minCapacity) {
        // overflow-conscious code
        //数组当前的长度（包括未填充的 这里最后注明一次 后边不写了）
        int oldCapacity = elementData.length;
        //扩容之后新数组的长度 = 现在数组的长度 + 现在数组长度除以2（这里用了>>运算，理论上要比/运算快的多） 即扩容之后的新数组长度为原来数组长度的1.5倍
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        //如果扩容1.5倍仍然比不上入参的数组大小 即新增元素之后的数组大小 那么就将扩容之后的长度变更成添加新元素之后的数组大小
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        //如果新数组容量比MAX_ARRAY_SIZE还大  那么就看一下添加元素之后的大小是不是比MAX_ARRAY_SIZE大
        //因为我们不可能无限扩容，我们指定了最大的长度为int最大值-8 
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        //扩容拷贝
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
```

整个方法的注释已经写在源码中，先获取现在的数组大小，然后扩容至1.5倍现在数组大小为新数组大小，然后判断这个新数组大小和添加完当前元素之后的数组大小哪个大，再判断这个新数组大小与MAX_ARRAY_SIZE比哪个大。可以看下MAX_ARRAY_SIZE的值为int的最大长度-8，emmm已经很大了撒。

```java
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
```

如果扩容之后比MAX_ARRAY_SIZE值大，那么我们就看一下添加元素之后的大小是多大。最后调用Arrays类的copyOf方法进行数组拷贝扩容。

#### 2.2.1.4hugeCapacity（int）

这里的入参是添加当前元素之后的大小，因为目前情况下，扩容之后的大小已经比指定的数组最大长度MAX_ARRAY_SIZE大了，所以我们有必要看一下添加完当前元素之后的长度是多少。

```java
    private static int hugeCapacity(int minCapacity) {
        //容量已经超了 内存溢出
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        //如果添加完当前元素之后的数组比MAX_ARRAY_SIZE大，那么就取Int的最大值 ，否则就取MAX_ARRAY_SIZE，而不是取之前计算之后的比MAX_ARRAY_SIZE还要大的扩容1.5倍之后的那个值
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }
```

这里相关内容已经进行了注释。

#### 2.2.1.5copyOf（Object[]，int）

这个方法是Arrays类的方法，具体内容不阐述了，看源码知道是用了System的arraycopy方法进行的拷贝，把原来的数组拷贝到扩容之后的数组中。arraycopy方法是一个Native方法

```java
    public static <T,U> T[] copyOf(U[] original, int newLength, Class<? extends T[]> newType) {
        @SuppressWarnings("unchecked")
        T[] copy = ((Object)newType == (Object)Object[].class)
            ? (T[]) new Object[newLength]
            : (T[]) Array.newInstance(newType.getComponentType(), newLength);
        System.arraycopy(original, 0, copy, 0,
                         Math.min(original.length, newLength));
        return copy;
    }
```

### 2.2.2add(int index, E element)

这个方法是向集合中指定位置添加元素。

```java
    public void add(int index, E element) {
        //检查插入的位置是否合法
        rangeCheckForAdd(index);
        //判断是否需要扩容
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        //因为在指定位置插入元素 所以后边的元素需要顺序后移（可以看出插入效率很低）
        System.arraycopy(elementData, index, elementData, index + 1,
                         size - index);
        //空出的位置插入指定元素
        elementData[index] = element;
        //已有元素长度+1
        size++;
    }
```

这个方法在指定位置插入指定元素，所以先要判断指定位置是否存在并合理，然后判断是不是需要扩容，然后将数组顺序向后移动，最后在空出的位置插入新元素之后已元素数组长度+1。

#### 2.2.2.1rangeCheckForAdd（int）

    private void rangeCheckForAdd(int index) {
        if (index > size || index < 0)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }
这个方法用来检查指定的位置是否合法，这里需要注意一点是，**插入的位置不能比当前数组长度大**。然后就是不能是负数。

判断扩容的方法这里就不赘述了，与上述相同，同样是修改次数加1然后判断与当前集合大小比较。

#### 2.2.2.2native void arraycopy(Object src,  int  srcPos，Object dest, int destPos,int length)

这个方法进行数组拷贝的，还是有必要说一下，当然是native方法，所以只说一下参数

```java
//src:源数组
//srcPos：源数组下标（从哪里开始拷贝）
//dest：新数组
//destPos：新数组下标（拷贝到新数组的起始位置）
//length：拷贝的数组长度（即指从源数组中拷贝多少到新数组）
public static native void arraycopy(Object src,  int  srcPos,
                                    Object dest, int destPos,
                                    int length);
```
五个参数说明注释在上述代码中，这里要做的是从要插入的位置的元素开始顺序向后移动一个位置，因为要给新来的元素腾位置。所以说实际就等同于从当前位置开始到末尾整体向后移动一个位置。

### 2.2.3boolean addAll(Collection<? extends E> c)

这个方法是向集合中添加指定集合中的全部元素。可以理解为两个集合拼接。

```java
    public boolean addAll(Collection<? extends E> c) {
        //将集合转成Object类型数组
        Object[] a = c.toArray();
        //新来了多长的数组
        int numNew = a.length;
        //判断新来这么多元素之后需不需要扩容
        ensureCapacityInternal(size + numNew);  // Increments modCount
        //把新来的数组复制到当前集合后边
        System.arraycopy(a, 0, elementData, size, numNew);
        //长度相应变化
        size += numNew;
        return numNew != 0;
    }
```

 这个方法没有什么特殊的地方，都是一些基本操作，就是把新来的数组拷贝到原来数组后边。

### 2.2.4boolean addAll(int index, Collection<? extends E> c)

这个方法是在指定位置添加指定的集合元素。就是前边的整合版，因此也比较简单。核心内容就是一个native方法完成的操作。

```java
public boolean addAll(int index, Collection<? extends E> c) {
    //检查下标
    rangeCheckForAdd(index);

    Object[] a = c.toArray();
    int numNew = a.length;
    //判断是否需要扩容
    ensureCapacityInternal(size + numNew);  // Increments modCount
	
    int numMoved = size - index;
    //是否需要移动元素（这里不可能<0 可以=0即变成了上一个方法 在当前集合之后插入新的集合）
    if (numMoved > 0)
        System.arraycopy(elementData, index, elementData, index + numNew,
                         numMoved);
	//将新插入的集合插到腾出的位置
    System.arraycopy(a, 0, elementData, index, numNew);
    size += numNew;
    return numNew != 0;
}
```
这个方法没有什么特殊的，主要是多了一个判断是否需要移动元素，可能是因为大量的集合移动比较浪费时间吧。

### 2.2.5小结

ArrayList一共有4种add的方法，分别为添加指定元素，添加指定集合，添加指定元素/集合到指定的位置。添加之前需要判断容量是否够用，否则需要进行扩容，扩容为当前大小的1.5倍。其数组拷贝的核心方法是native方法System.arrayCopy（），每一次新增操作修改次数参数modCount都会自增。

## 2.3从ArrayList中获取元素

### 2.3.1E get(int index)

这个方法是从集合中获取单个元素，ArrayList也只能获取单个元素。

```java
public E get(int index) {
	//下标检查
    rangeCheck(index);
	//返回指定位置的元素
    return elementData(index);
}
```
这里没有什么特别的地方，就是先判断位置是否合法，然后返回指定元素。

#### 2.3.1.1rangeCheck（int）

下标合法性校验

```java
private void rangeCheck(int index) {
    if (index >= size)
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}
```
这里即便不校验，数组本身也会抛出异常。

#### 2.3.1.2elementData（index）

返回数组中指定下标的元素

```java
E elementData(int index) {
    return (E) elementData[index];
}
```
### 2.3.2小结

ArrayList提供了一个get（int index）方法返回指定下标位置的元素。

## 2.4从ArrayList中删除元素

### 2.4.1E remove(int index)

该方法删除指定下标上的元素，并返回该元素。

```java
public E remove(int index) {
    //检查下标合法性
    rangeCheck(index);
	//修改次数+1
    modCount++;
    //记录要删除的元素
    E oldValue = elementData(index);
	//要移动元素个数
    int numMoved = size - index - 1;
    //如果要移动的个数>0 即不是删除的最后一个元素
    if (numMoved > 0)
        //将要删除的元素下一个元素起后边的所有元素整体前移一位（整体拷贝）
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    //把最后一个元素赋值为null并把已有元素容量-1
    elementData[--size] = null; // clear to let GC do its work
	//返回删除的元素
    return oldValue;
}
```
这个方法也都是一些基本操作，通过这个方法可以看到**删除指定位置的元素实际上是把它后边的元素整体拷贝前移一位，然后将最后的元素赋值为null删除。**

### 2.4.2 boolean remove(Object o)

这个方法是删除指定元素，并返回成功或者失败。

```java
public boolean remove(Object o) {
    //ArrayList允许一个位置的元素值为null 即==null
    if (o == null) {
        //遍历数组找到第一个 == null的下标
        for (int index = 0; index < size; index++)
            if (elementData[index] == null) {
                //删除该下标的元素
                fastRemove(index);
                return true;
            }
    } else {
        //遍历数组找到第一个 equals o的下标
        for (int index = 0; index < size; index++)
            if (o.equals(elementData[index])) {
                //删除该下标的元素
                fastRemove(index);
                return true;
            }
    }
    //没有遍历到则返回false
    return false;
}
```
这个方法是拿着给定的元素进行遍历，遍历分为两种，一种是==null，一种是不为null的时候用equals（）方法。然后找到下标之后调用faseRemove（index）方法移除指定位置的元素。如果没有遍历到则返回false。

#### 2.4.2.1fastRemove（int）

这个方法实际就是remove（int index）方法中的删除方法，1.7之前remove（int index）方法也是调用的fastRemove（），这里不知道为什么改了，但是方法内容还都是一样的。

```java
private void fastRemove(int index) {
    //修改次数+1
    modCount++;
    //要移动的元素个数
    int numMoved = size - index - 1;
    if (numMoved > 0)
        //拷贝要删除的元素之后的元素块向前移动一位
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    //移除最后一个元素并更改大小
    elementData[--size] = null; // clear to let GC do its work
}
```
### 2.4.3boolean removeAll(Collection<?> c)

这个方法删除ArrayList中指定集合中存在的元素

```java
public boolean removeAll(Collection<?> c) {
    //确认传入的集合不为空
    Objects.requireNonNull(c);
    //批量删除
    return batchRemove(c, false);
}
```
先调用Objects类的判断方法判断入参是否为null，

#### 2.4.3.1boolean batchRemove(Collection<?> c, boolean complement)

这个方法用来批量移除指定集合中存在的元素

```java
private boolean batchRemove(Collection<?> c, boolean complement) {
    //当前ArrayList集合指向一个新的引用 （修改会改变原ArrayList）个人感觉这个步骤可能是为了写起来方便？不用一直this.elementData？
    final Object[] elementData = this.elementData;
    int r = 0, w = 0;
    //是否修改成功
    boolean modified = false;
    try {
        //遍历ArrayList集合，如果当前元素不在要删除的集合中 则把第r个元素填充到第w个位置 然后w+1 否则只有r+1 可以理解为w为遍历之后去掉删除的元素剩余的元素个数
        for (; r < size; r++)
            if (c.contains(elementData[r]) == complement)
                elementData[w++] = elementData[r];
    } finally {
        // Preserve behavioral compatibility with AbstractCollection,
        // even if c.contains() throws.
        //正常情况下这里r一定==size 当c.contains出现异常时，r将！=size 这里为了保证一致性，将后边没遍历完的数组填充到w后  w为新数组的size 因此需要加上未遍历的个数
        if (r != size) {
            //把遍历剩下的数组拷贝w之后的位置
            System.arraycopy(elementData, r,
                             elementData, w,
                             size - r);
            w += size - r;
        }
        //如果有元素删除了
        if (w != size) {
            // clear to let GC do its work
            //说明删除之后数组长度只到w 而实际长度为size 需要把多余的删除
            for (int i = w; i < size; i++)
                //删除长度为w之后的个数
                elementData[i] = null;
            //修改次数+删除次数
            modCount += size - w;
            //长度为w
            size = w;
            //修改成功
            modified = true;
        }
    }
    //返回修改成功还是失败
    return modified;
}
```
这个方法有点长，相对有一点点绕。首先定义了一个r和w变量，r变量用来遍历原来的数组，w变量用来记录新数组的长度。入参的complement可以译为是否保存。然后看逻辑，首先是try内部的逻辑：

```java
        for (; r < size; r++)
            if (c.contains(elementData[r]) == complement)
                elementData[w++] = elementData[r];
```
用变量r遍历原来的数组，如果遍历道的元素在要删除的数组中，则不保留，否则保留。这里做一个示例逻辑演示一下：

假设我们现在的ArrayList数组1是1,2,3,4,5 共五个元素，我们要删除的集合数组2是3,5,6,7,8五个元素。那么我们遍历一下这段代码会出现什么情况。

1. r=0,w=0时，因为元素1不包含在数组2中，所以执行 elementData[w++] = elementData[r]，此时数组1为1,2,3,4,5 r =1,w=1
2. r=1,w=1时，因为元素2不包含在数组2中，所以执行 elementData[w++] = elementData[r]，此时数组1为1,2,3,4,5 r =2,w=2
3. r=2,w=2时，因为元素3包含在数组2中，所以不执行后边的逻辑，此时数组1为1,2,3,4,5 r=3,w=2
4. r=3,w=2时，因为元素4不包含在数组2中，所以执行 elementData[w++] = elementData[r]，此时数组1为1,2,4,4,5 r =4,w=3
5. r=4,w=3时，因为元素5包含在数组2中，所以不执行后边的逻辑，此时数组1为1,2,4,4,5 r=5，w=3

我们可以看到，实际上就是遍历如果不是要删除的元素就保留下来按顺序覆盖原来的数组。此时w=3，即新数组的size=3

接下来看finally中的方法

```java
        // Preserve behavioral compatibility with AbstractCollection,
        // even if c.contains() throws.
        if (r != size) {
            System.arraycopy(elementData, r,
                             elementData, w,
                             size - r);
            w += size - r;
        }
        if (w != size) {
            // clear to let GC do its work
            for (int i = w; i < size; i++)
                elementData[i] = null;
            modCount += size - w;
            size = w;
            modified = true;
        }
```
这里因为是用r遍历的数组size，所以理论上讲r一定是会等于size的。不过在循环中c.contains（）方法可能会出现异常，因此可能会出现r！=size的情况，这时我们为了保持一致性，需要将未遍历的元素拷贝到w之后。然后修改w的长度加上未遍历的元素个数。如果w即新数组的长度不等于size，那么我们就需要把多出来的元素删除。也就是遍历w之后到size的那部分元素删除，然后修改次数加上删除的次数，最后新数组的size就是w了。

### 2.4.4小结

从ArrayList中删除元素有三种方法，一种是删除指定下标上的元素，一种是删除指定元素，最后一种是删除指定集合内存在的元素。删除操作与新增操作基本核心都是用了arraycopy方法移动数组块，因此理论上讲效率很低。而且remove（int）不需要遍历整个集合，而remove（Object）需要遍历集合寻找第一个符合条件的元素。remove支持移除元素==null的情况。removeAll的原理是遍历原数组找到不在要删除的集合中的元素保留下来然后删除多余的那一部分长度的数组，在调用c.contains（）方法时可能出现异常。

## 2.5判断一个元素在集合中是否存在

### 2.5.1boolean contains(Object o)

该方法用于判断一个对象是否在集合中，存在返回true，否则返回false

```java
    public boolean contains(Object o) {
        //返回该元素的下标 存在则>=0 否则返回-1
        return indexOf(o) >= 0;
    }
```

#### 2.5.1.1int indexOf(Object o)

该方法返回元素在数组中的下标，不存在返回-1，支持null==判断

```java
public int indexOf(Object o) {
    if (o == null) {
        for (int i = 0; i < size; i++)
            if (elementData[i]==null)
                return i;
    } else {
        for (int i = 0; i < size; i++)
            if (o.equals(elementData[i]))
                return i;
    }
    return -1;
}
```
#### 2.5.1.2int lastIndexOf(Object o) 

ArrayList还提供了获取匹配元素的最后一个下标，即上述方法倒叙for循环

```java
public int lastIndexOf(Object o) {
    if (o == null) {
        for (int i = size-1; i >= 0; i--)
            if (elementData[i]==null)
                return i;
    } else {
        for (int i = size-1; i >= 0; i--)
            if (o.equals(elementData[i]))
                return i;
    }
    return -1;
}
```
### 2.5.2小结

ArrayList提供了contains方法判断是否存在指定对象元素，该方法通过indexOf方法返回第一个出现指定元素的下标，同时也提供了返回最后一个出现指定元素的下标。

## 2.6ArrayList中的迭代器

迭代器主要作用是用来遍历集合，基本使用形式如下：

```java
package com.chenxyt.threadTest;
import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;
public class BugTest {
	public static void main(String[] args) {
	    List<String> list = new ArrayList<String>();
        list.add("chenxyt");
        list.add("yang");
        list.add("nina");
        Iterator<String> it = list.iterator();
        while (it.hasNext()) {
            System.out.println(it.next());
        }
	}
}

```

获取一个集合的迭代器对象，然后通过这个迭代器进行对象的遍历。

### 2.6.1iterator（）

这个方法返回一个迭代器对象

```java
public Iterator<E> iterator() {
    return new Itr();
}
```
Iterator是一个泛型接口，Itr是它的一个实现类。

### 2.6.2Itr

Itr是ArrayList的一个内部类，实现了迭代器Iterator接口，这里贴出类的具体内容：

```java
/**
     * An optimized version of AbstractList.Itr
     */
//AbsractList.Itr的最佳版本    Itr原来的版本是AbstractList的内部类 1.8之后放到了ArrayList中
    private class Itr implements Iterator<E> {
        //记录该遍历的元素下标
        int cursor;       // index of next element to return
        //记录上次返回的下标 如果没有返回默认-1
        int lastRet = -1; // index of last element returned; -1 if no such
        //记录修改的次数 --这个很重要 用来判断在迭代器遍历过程中ArrayList是否发生了修改
        int expectedModCount = modCount;
        //是否有下一个元素可以遍历 size为元素个数 cursor是下一个要遍历的下标 所以最大的cursor=size-1
        //如果他俩相等了则表示遍历到最后了
        public boolean hasNext() {
            return cursor != size;
        }
		//返回下一个元素
        @SuppressWarnings("unchecked")
        public E next() {
            //检查遍历过程是否发生了修改
            checkForComodification();
            //拿到当前遍历的元素下标
            int i = cursor;
            //下标只能<size才有数据
            if (i >= size)
                throw new NoSuchElementException();
            //拿到ArrayList集合内部的数组对象
            Object[] elementData = ArrayList.this.elementData;
            //同理i也不能比length大（length==size）  因为前边判断过一次 所以这里如果比length大则表示
            //在迭代器遍历的过程 ArrayList发生了list.remove（）操作
            if (i >= elementData.length)
                throw new ConcurrentModificationException();
            //记录下次要遍历的下标
            cursor = i + 1;
            //返回当前下标元素 并把返回的下标标记
            return (E) elementData[lastRet = i];
        }
	    //迭代器的移除上次返回的元素 即cursor-1 即 lastRet 移除元素只能移除已经遍历且返回的那个
        public void remove() {
            //没有返回过
            if (lastRet < 0)
                throw new IllegalStateException();
            //检查移除过程是否发生了修改
            checkForComodification();
			
            try {
                //调用集合的remove方法 移除lastRet下标上的元素
                ArrayList.this.remove(lastRet);
                //下次要遍历的下标-1（因为移除了遍历返回过的最末元素）
                cursor = lastRet;
                //重置返回标记
                lastRet = -1;
                //发生了移除操作 重新获取修改次数modCount
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }
		
        @Override
        @SuppressWarnings("unchecked")
        //jdk8提供的快速遍历方式 与foreach基本相同
        public void forEachRemaining(Consumer<? super E> consumer) {
            //判断传入的Consumer对象是否为空
            Objects.requireNonNull(consumer);
            //获取当前集合的size
            final int size = ArrayList.this.size;
            //遍历的位置 如果没遍历过则初始化为0
            int i = cursor;
            if (i >= size) {
                return;
            }
            //获取集合的数组对象
            final Object[] elementData = ArrayList.this.elementData;
            //是否发生过修改
            if (i >= elementData.length) {
                throw new ConcurrentModificationException();
            }
            //遍历 并且外部未发生修改时 执行accept重写方法的操作 比如print、remove等
            while (i != size && modCount == expectedModCount) {
                consumer.accept((E) elementData[i++]);
            }
            // update once at end of iteration to reduce heap write traffic
            //更新下一个要遍历的数组下标
            cursor = i;
            //重置lastRest --这里结合重写的accept方法会可能会发生异常 后边单独说
            lastRet = i - 1;
            //检查遍历过程中是否发生了修改
            checkForComodification();
        }
		//检查遍历过程中是否发生了移除/插入操作
        final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
    }
```

相关代码上边都有注释，下面针对具体的方法进行详细的说明。

### 2.6.3boolean hasNext（）

这个方法是判断当前迭代器是否还可以继续遍历下一个元素

```java
    public boolean hasNext() {
        //cursor是当前迭代器下一次要遍历到的元素下标 没遍历一个元素 cursor+1  因为数组长度为size  而最         //大下标为size-1 所以当cursor=size时  即不可遍历了
        return cursor != size;
    }
```
这个方法的核心是返回是否可以遍历“**下一个**”元素。

### 2.6.4E next（）

这个方法用来获取下一个元素

```java
    @SuppressWarnings("unchecked")
    public E next() {
        //判断在遍历的过程中 是否有元素发生过修改
        checkForComodification();
        //获取当前要获取的元素下标
        int i = cursor;
        if (i >= size)
            throw new NoSuchElementException();
        //获取当前数组
        Object[] elementData = ArrayList.this.elementData;
        //如果要遍历获取的下标比当前长度大  则说明发生过remove操作
        if (i >= elementData.length)
            throw new ConcurrentModificationException();
        //下一个要遍历的下标
        cursor = i + 1;
        //返回数组中指定下标位置的元素  并记录本次返回的下标
        return (E) elementData[lastRet = i];
    }
```
这个方法用来返回迭代器中遍历的下一个元素。即与hasNext（）配合使用，如果hasNext返回了true，则调用next方法可以返回下个元素。

### 2.6.5checkForComodification（）

这个方法用来检查在遍历的过程中是否发生过修改

```java
    final void checkForComodification() {
        //前边可以看到 在外部类中 如果发生了增删操作 modCount会++ 而我们在迭代器内部 初始化的时候会将expectedModCount的值赋值成modCount  只有在调用了迭代器内部的remove操作之后会变更一次这个值
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
    }
```
如注释所说，expectedModCount这个值在初始化的时候会赋值，在外部被修改的时候，这里是不会做变更，只有在迭代器内部调用了remove方法之后才会进行变更。这个方法抛出的异常ConcurrentModificationException是一个经典异常。后边章节针对这个异常具体说一下。

### 2.6.6remove（）

迭代器自带了一个remove方法，删除对应数组上的元素。

```java
    public void remove() {
        //lastRet只有在进行过next（）操作也就是遍历过元素之后才会>0 因此可以猜到这里remove操作remove的是之前遍历过的元素 而不能像外部类那样指定下标和元素
        if (lastRet < 0)
            throw new IllegalStateException();
        //检查是否发生过改变
        checkForComodification();
        try {
            //移除刚才遍历过的元素
            ArrayList.this.remove(lastRet);
            //删除操作会把后边的元素向前补一位 因此下一次遍历的仍然是刚才删除的那一位
            cursor = lastRet;
            //没有遍历过的元素了 下标改成-1
            lastRet = -1;
            //因为调用了remove方法 所以modCount改变了 这里重新赋值
            expectedModCount = modCount;
        } catch (IndexOutOfBoundsException ex) {
            throw new ConcurrentModificationException();
        }
    }
```
### 2.6.7forEachRemaining(Consumer<? super E> consumer) 

这个方法是jdk1.8之后的一个方法，传入一个Consumer对象用来遍历实现相关逻辑，与for-each功能基本相同

```java
        public void forEachRemaining(Consumer<? super E> consumer) {
            //判断传入的对象是否为空
            Objects.requireNonNull(consumer);
            //获取当前数组的长度
            final int size = ArrayList.this.size;
            //当前遍历的位置（可能在forEachRemaining方法之前调用过其它遍历方法）
            int i = cursor;
            //i比size大则不需要进行遍历了
            if (i >= size) {
                return;
            }
            //获取当前数组
            final Object[] elementData = ArrayList.this.elementData;
            //如果i比当前数组长度大 则说明发生过修改
            if (i >= elementData.length) {
                throw new ConcurrentModificationException();
            }
            //继续遍历当前迭代器 并且保证在外部未发生过修改的情况下 执行Consumer对象重写的accept方法
            while (i != size && modCount == expectedModCount) {
                consumer.accept((E) elementData[i++]);
            }
            // update once at end of iteration to reduce heap write traffic
            //假设外部发生过修改 则上述while循环会在遍历完数组之前提前退出 因此需要将下一个要遍历的值赋给cursor
            cursor = i;
            //上一个返回的就是i-1 这里如果重写方法中有remove操作 会发生异常
            lastRet = i - 1;
            //检查是否发生了修改
            checkForComodification();
        }
```

### 2.6.8forEachRemaining（）引发的异常

上个小节说到使用forEachRemaining（）方法进行遍历时，可能会出现异常。这里通过一个demo分析为何会产生异常，同时也看下forEachRemaining（）方法怎么使用。

```java
package com.chenxyt.threadTest;
import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;
import java.util.function.Consumer;
public class BugTest {
	public static void main(String[] args) throws InterruptedException {
		ArrayList<String> list = new ArrayList();
        for (int i = 0; i < 10; i++) {
            list.add(String.valueOf(i));
        }
        Iterator iterator = list.iterator();
        iterator.forEachRemaining(new Consumer() {
            @Override
            public void accept(Object o) {
                System.out.println(o);
                  if (o.equals("3") ) {
                      System.out.println("remove");
                      iterator.remove();
                  }
            }
        });
	}
}
```

如上，就是遍历然后在accept方法内部，完成迭代器的操作。这里使用迭代器进行remove（）操作。

运行结果：

![1554821208339](【源码学习】Java集合之ArrayList\1554821208339.png)

可以看到在执行remove之前抛出了异常-java.lang.IllegalStateException，根据堆栈信息可以看到是在remove方法抛出的异常。再回头看一下迭代器的remove方法。

```java
        public void remove() {
            if (lastRet < 0)
                throw new IllegalStateException();
            checkForComodification();

            try {
                ArrayList.this.remove(lastRet);
                cursor = lastRet;
                lastRet = -1;
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }
```

可以看到这个异常是在判断lastRet<0之后返回的。我们知道lastRet是与next（）方法配合使用的，每用next（）方法返回一个遍历元素之后，lastRet赋值为返回的元素下标。而在forEachRemaining（）方法中，遍历元素是通过while循环一个一个遍历的，这期间并没有修改lastRet的值，因此这里如果直接调用了remove（）方法就会抛出java.lang.IllegalStateException.

### 2.6.9经典异常ConcurrentModificationException

这个异常不光在ArrayList中会出现，在其它使用迭代器的集合中也会出现。下面一个示例演示这个异常。

```java
package com.chenxyt.threadTest;
import java.util.ArrayList;
public class BugTest {
	public static void main(String[] args) throws InterruptedException {
		ArrayList<String> list = new ArrayList();
        for (int i = 0; i < 10; i++) {
            list.add(String.valueOf(i));
        }
        for(String s:list){
        	System.out.println(s);
        	if("3".equals(s)){
        		System.out.println("remove");
        		list.remove("3");
        	}
        }
	}
}

```

这个demo通过一个for-each增强for循环完成了遍历并使用ArrayList外部类的remove（Object）方法删除指定元素。运行结果如下：

![1554821894511](【源码学习】Java集合之ArrayList\1554821894511.png)

可以看到发生了java.util.ConcurrentModificationException，因为for-each增强for循环的原理实际就是迭代器hasNext（）和next（）方法的循环使用。而迭代器中exceptedModCount只有初始化的时候赋值为modCount和调用迭代器内部的remove（）方法之后赋值为modCount，其它时候并没有改变。因此在遍历的过程中如果调用了ArrayList外部类的remove（）方法会导致modCount发生改变，从而在进行next（）方法checkForComodification（）方法之后抛出这个异常。

通过javap -c 可以看到这个class文件的字节码中使用了迭代器中hasNext（）和next（）等方法

![1554822461669](【源码学习】Java集合之ArrayList\1554822461669.png)

### 2.6.10删除重复字符会出现的陷阱

假设我们有个ArrayList数组，其中有两位连着的元素是一样的，我们想删除这两个元素。有如下示例：

```java
package com.chenxyt.threadTest;
import java.util.ArrayList;
public class BugTest {
	public static void main(String[] args) throws InterruptedException {
		ArrayList<String> list = new ArrayList();
        for (int i = 0; i < 5; i++) {
            list.add(String.valueOf(i));
            if(i == 3){
                list.add(String.valueOf(i));
            }
        }
        System.out.println("删除前集合元素");
        for(String s : list){
        	System.out.println(s);
        }     
        System.out.println("------");
        for(int i = 0;i<list.size();i++){
        	if(list.get(i).equals("3")){
                 System.out.println("删除下标为：" + i + "的元素"); 
        		list.remove(i);
        	}
        }
        System.out.println("删除后集合元素");
        for(String s : list){
        	System.out.println(s);
        }     
	}
}
```

这个demo构造了一个带有重复元素“3”的ArrayList集合，然后不适用迭代器遍历删除重复的元素“3”，看一下运行结果：

![1554823008905](【源码学习】Java集合之ArrayList\1554823008905.png)

哦豁，只删除了下标为3的元素，而重复的元素即下标为4的那个“3”没有被删除成功。这里出现这个情况的原因是因为ArrayList的remove操作会把删除位置后边的元素向前移动，原来下标为4的那个“3”在下标为3的那个“3”被删除之后向前移动了一位变成了下标3，这时候的i已经继续循环+1了，因此无法将后边重复的那个“3”删除。

### 2.6.11小结

ArrayList的迭代器还是相对重要些，主要用于遍历和删除操作。要注意exceptModCount的值，避免在使用迭代器遍历时调用外部的增删方法，以免发生ConcurrentModificationException异常。

## 2.7小结

1. ArrayList是通过transient Object数组构造的一个容器，数组的特性就是查询快，可以达到O(1)时间复杂度，而新增删除代价比较大，因为涉及到了元素位置的移动。构造方法可以初始化指定长度的容器，也可以初始化指定元素的容器，默认的无参构造函数初始化空数组，在插入第一个元素之后会扩容为10
2. 添加元素的时候会判断容量大小，添加完如果长度比当前数组的容量大，则扩容为当前容量的1.5倍，扩容之后使用native arraycopy方法进行数组拷贝，添加操作会修改modCount的值
3. 获取元素直接调用get方法，返回指定下标的元素
4. 删除元素调用fastRmove（）进行删除，删除指定位置的元素或者指定元素，可以删除为null的元素，删除的过程是将要删除的元素后边一位到最后一位向前拷贝一位，然后删除最后的一位元素，删除操作会修改modCount的值
5. contains方法返回一个元素是否在集合中，是通过indeOf（）即遍历查找指定元素返回其下标与0比较，小于0则返回false不存在，否则即存在
6. ArrayList中的迭代器用于遍历操作，在迭代器初始化的时候会将此时的modCount赋值给exceptCount，因此如果在遍历的过程中发生了外部类remove操作，修改了modCount则会抛出ConcurrentModificationException异常。
7. jdk1.8中新增的forEachRemaining方法在遍历的过程是一个一个向后移动下标，没有像迭代器的hasNext/next操作返回lastRet值，因为调用迭代器内部的remove方法时会发生异常，因为lastRet可能为-1
8. 不使用迭代器遍历ArrayList时，删除相邻重复元素不会成功，因为ArrayList.remove方法是把后边的元素前移了，重复元素的下标变了。

