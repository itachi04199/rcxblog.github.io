---
layout: post
title: Java 解惑 -- 表达式谜题
categories: Java 解惑
tags: 笔记
---

#### 奇数性

在判断一个整数是否为奇数的时候不要用余数为1来判断，因为会有负数的情况，应该用余数不为0来判断。如果需要更加的性能，可以使用如下的方式：

```java
public static boolean isOdd(int i){
	return (i & 1) != 0;
}
```

#### 精度运算

涉及到精度运算的不要使用 double 类型，因为 double 类型不是精确的，应该使用 BigDecimal。

并且一定要用 BigDecimal(String) 构造器，而千万不要用BigDecimal(double)。

后一个构造器将用它的参数的“精确”值来创建一个实例：

```java
new BigDecimal(.1);

将返回一个表示0.100000000000000055511151231257827021181583404541015625的 BigDecimal。
```

#### 长整数

在对长整数值的运算过程中，需要注意的是在运算当中添加长整型标志。如下这样，如果不加 `L` ,就会整数溢出，并且不要加小写的，因为很容易跟数字一混乱。

```java
final long MICROS_PER_DAY = 24L * 60 * 60 * 1000 * 1000;
final long MILLIS_PER_DAY = 24L * 60 * 60 * 1000;
```

#### 多重转型

看如下例子会打印出什么：

```java
public class Multicast{
	public static void main (String[] args){
		System.out.println((int)(char)(byte) -1);
	}
}
```

它打印出来的是65535。

从 int 到 byte 的转型是很简单的，它执行了一个窄化原始类型转化，直接将除低 8 位之外的所有位全部砍掉。

如果最初的数值类型是有符号的，那么就执行符号扩展；如果它是char，那么不管它将要被转换成什么类型，都执行零扩展。

从 byte 到 char 的转型稍微麻烦一点，因为 byte 是一个有符号的类型，所以在将 byte 数值 -1 转换成 char 时，会发生符号扩展。

#### 三重运算符的返回值类型

```java
public class DosEquis{
	public static void main(String[] args){
        char x = 'X';
        int i = 0;
        System.out.println(true ? x : 0);
        System.out.println(false ? i : x);
	}
}
```

什么这段代码的输出是：

```java
X
88
```

每一个表达式的第二个和第三个操作数的类型都不相同，有如下规则：

- 如果第二个和第三个操作数具有相同的类型，那么它就是条件表达式的类型。
- 如果一个操作数的类型是T，T 表示 byte、short 或 char，而另一个操作数是一个 int 类型的常量表达式，它的值是可以用类型 T 表示的，那么条件表达式的类型就是 T。
- 否则，将对操作数类型运用二进制数字提升，而条件表达式的类型就是第二个和第三个操作数被提升之后的类型。

根据这个规则就可以解释为什么这两个表达式的值输出的不一样了，因为 println 方法调用的是不同的。一个是接受 char 参数的，一个是接受 int 参数的。

#### 复合赋值

复合赋值操作符包括

```java
+=、-=、*=、/=、%=、<<=、>>=、>>>=、&=、^= 、|=
```

Java 语言规范中讲到，复合赋值 E1 op= E2 等价于简单赋值 E1 = (T)((E1)op(E2))，其中 T 是E1 的类型，除非 E1 只被计算一次。

为了避免这种情况，请不要将复合赋值操作符作用于 byte、short 或char 类型的变量上。

在将复合赋值操作符作用于 int 类型的变量上时，要确保表达式右侧不是 long、float 或 double 类型。

在将复合赋值操作符作用于 float 类型的变量上时，要确保表达式右侧不是 double 类型

【参考资料】

1.  [java解惑](http://book.douban.com/subject/1473329/)

---EOF---


