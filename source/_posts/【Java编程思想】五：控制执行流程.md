---
title: 【Java编程思想】五：控制执行流程
date: 2018-11-13 09:48:20
tags:
- Java 
- Think in Java
categories: 学习笔记
---

# 一、true和false

&ensp;&ensp;&ensp;&ensp;关系操作符构造的条件语句如“==”的返回值是true和false，Java中不允许将一个数字作为布尔值使用。

# 二、if-else

&ensp;&ensp;&ensp;&ensp;if else语句与其它语言的相同，其中else是可选的。if else用来实现多种条件下的执行。

# 三、迭代

&ensp;&ensp;&ensp;&ensp;while、do-while、for用来控制循环，有时候将他们称为迭代语句。语句会重复执行，直到起控制作用的布尔值得到“假”的结果时停止

# 四、for-each

&ensp;&ensp;&ensp;&ensp;for-each是一种更加简洁的for语句，语法如下：

```java
for(int i:x){
    //---
}
```

&ensp;&ensp;&ensp;&ensp;x是要被访问的循环体，i是一个变量，类型int只是一种，具体的类型要与x内部的值的类型相同，这个语句的意思就是循环取x内部的值赋值给i

# 五、return

&ensp;&ensp;&ensp;&ensp;return关键字有两个用途，一方面指定一个方法返回什么值，另一方面它会强制结束当前方法，并返回那个值。

# 六、break和continue

&ensp;&ensp;&ensp;&ensp;在任何迭代语句的主体部分，都可以用break和continue控制循环的流程，其中break用于强制退出循环，不执行循环剩余的语句，比如一共五组数据循环到第三组break，那么后面两组不管了继续执行下边的数据。continue是停止当前的迭代，退到循环开始执行下一次迭代，比如一共五组数据执行到第三组开始的时候continue，那么这个循环体主体剩余的部分不执行，继续从第四组开始执行。

# 七、臭名昭著的go-to

&ensp;&ensp;&ensp;&ensp;go-to语句会破坏代码的逻辑结构，降低代码的可读性。因此不建议使用。

# 八、switch

 &ensp;&ensp;&ensp;&ensp;switch也被划为一种选择语句，根据整数表达式的值，从一系列语句中选择一组执行。语法如下：

```java
switch(a){
    case value1:
       //---;
        break;
    case value2:
        //---;
        break;
    default:
        //---;
}
```

# 九、总结

&ensp;&ensp;&ensp;&ensp;文中多数控制语句在其它语言中都通用，注意for-each语句的使用。