---
layout: post
title: 程序设计风格
categories: 代码优化
tags: 笔记 代码优化 程序设计实践笔记
---

### 名字

什么是名字？一个变量或函数的名字标识这个对象，带着说明其用途的一些信息。一个名
字应该是非形式的、简练的、容易记忆的。许多信息来自上下文和作用范围(作用域)。一个变量的作用域越大，它的名字所携带的信息就应该越多。

全局变量使用具有说明性的名字，局部变量用短名字。根据定义，全局变量可以出现在整个程序中的任何地方，因此它们的名字应该足够长，具有足够的说明性，以便使读者能够记得它们是干什么用的。

人们常常鼓励程序员使用长的变量名，而不管用在什么地方。这种认识完全是错误的，清晰性经常是随着简洁而来的。例如用i、j作为循环变量，p、q作为指针，s、t表示字符串等。

**保持一致性**，相关的东西名字应该保持一致性，如下的代码不是很好。

```java
class UserQueue {
	int noOfItemsInQ, frontOfQueue, queueCapacity;
    public int noOfUserInQueue(){...}
}
```

这里同一个词“队列(queue)”在名字里被分别写为Q、Queue或queue。由于只能在类型UserQueue里访问，类成员的名字中完全不必提到队列，因为存在上下文。所以修改成下面这样会更好：

```java
class UserQueue {
	int itemsIndex, front, capacity;
    public int userIndex(){...}
}
```

实际上item和user说明的是一个东西，所以也可以把itemsIndex修改成userIndex。


**函数采用动作性的名字**。函数名应当用动作性的动词，后面可以跟着名词：

```java
now = date.getTime();
```

对于返回布尔值的函数，应该清楚说明其返回值。如下：

```java
if(checkoctal(a)){...}

if(isoctal(a)){...}
```

相比之下还是下面的命名比较清晰的知道函数的返回值。

### 表达式和语句

用缩行显示程序的结构。采用一种一致的缩行风格，是程序呈现出结构清晰的最省力的方法。

使用表达式的自然形式。表达式应该写得你能大声念出来。含有否定运算的条件表达式比较难理解：

```java
if(!( a < b) || !(a >= c)) {...}
```

这两个表达式当中都用到了否定，而且他们都不是必要的。应该改变运算符的方向，使表达式变成肯定。

```java
if((a >= b) || (a < c)) {...}
```

用加括号的方式排除二义性。括号表示分组，即使有时并不必要，加了括号也可能把意图表示得更清楚。

### 一致性和习惯用法

为了一致性，使用习惯用法。和自然语言一样，程序设计语言也有许多惯用法，也就是那些经验丰富的程序员写常见代码片段的习惯方式。在学习一个语言的过程中，一个中心问题就是逐渐熟悉它的习惯用法。

for循环的经典写法：

```java
for(int i = 0; i < n; i++) {
	array[i] = i;
}
```

下面是在c语言当中扫描一个链表的标准写法：

```c
for(p = list; p != NULL; p = p -> next) {
	...
}
```

对于无限循环我们经常使用：

```java
for(;;){
	...
}
```

### 神秘的数(Magic Numbers)

神秘的数包括各种常数、数组的大小、字符位置、变换因子以及程序中出现的其他以文字形式写出的数值。

给神秘的数起个名字。作为一个指导原则，除了0和1之外，程序里出现的任何数大概都可以算是神秘的数，它们应该有自己的名字。

在代码当中常用的方式是将神秘的数跟enum关联上，或者在Java中也可以定义成常量。

### 注释

注释应该提供那些不能一下子从代码中看到的东西，或者把那些散布在许多代码里的信息收集到一起。当某些难以捉摸的事情出现时，注释可以帮助澄清情况。如果操作本身非常明了，重复谈论它们就是画蛇添足了。

给函数和全局数据加注释。注释当然可以有价值。对于函数、全局变量、常数定义、结构和类的域等，以及任何其他加上简短说明就能够帮助理解的内容，我们都应该为之提供注释。

**不要注释差的代码，重写它。**

注释不要与代码矛盾。许多注释在写的时候与代码是一致的。但是后来由于修正错误，程序改变了，可是注释常常还保持着原来的样子，从而导致注释与代码的脱节。

**我们应该尽可能地把代码写得容易理解。在这方面你做得越好，需要写的注释就越少。好的
代码需要的注释远远少于差的代码。**

【参考资料】

1. 《程序设计实践》


---EOF---

