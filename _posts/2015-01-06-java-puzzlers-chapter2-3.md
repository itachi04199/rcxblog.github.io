---
layout: post
title: Java 解惑 -- 字符串、循环谜题
categories: Java解惑
tags: 笔记
---

#### 字符串连接

看下面程序会打印出什么：

```java
public class LastLaugh{
    public static void main(String[] args){
        System.out.print("H"+"a");
        System.out.print('H'+'a');
    }
}
```

上面的程序会输出 Ha169。

'H' 和 'a' 是字符型字面常量，因为这两个操作数都不是字符串类型的，所以 + 操作符执行的是加法而不是字符串连接。

总之，使用字符串连接操作符使用格外小心。+ 操作符当且仅当它的操作数中至少有一个是 String 类型时，才会执行字符串连接操作；否则，它执行的就是加法。

如果要连接的没有一个数值是字符串类型的，那么你可以有几种选择：

- 预置一个空字符串；
- 将第一个数值用 String.valueOf 显式地转换成一个字符串；
- 使用一个字符串缓冲区；
- 或者如果你使用的 JDK 的 printf 方法。

#### 运算符顺序

看下面代码会输出什么：

```java
public class AnimalFarm{
    public static void main(String[] args){
        final String pig = "length: 10";
        final String dog = "length: " + pig.length();
        System.out. println("Animals are equal: "
                            + pig == dog);
    }
}
```

上面的代码只会输出 false。这可能很出人意料，但是仔细分析下，执行的过程是这个样子的。

首先会先进行 + 操作，然后在进行 `== `操作，所以一定会返回 false。这是因为 + 操作符的优先级高于 `==` 操作符。

下面是 java 操作符优先级表：

类别	|操作符	|关联性
:--|:--|:--
后缀|	() [] . (点操作符)	|左到右
一元|	+ + - ！〜	|从右到左
乘性| 	* / ％	|左到右
加性| 	+ -	|左到右
移位| 	>>、>>> 、<< 	|左到右
关系| 	>、>=、 <、<= 	|左到右
相等| 	== 、!=	|左到右
按位与|	＆	|左到右
按位异或|	^	|左到右
按位或|　丨	|左到右
逻辑与|	&&	|左到右
逻辑或| 丨丨	|左到右
条件|	？：	|从右到左
赋值|	=、 +=、 -=、 *=、 /=、％=、 >>=、 <<=、＆=、 ^ =、 丨=	|从右到左
逗号|	，	|左到右

#### 增量操作

看下面程序的输出结果：

```java
public class Increment {
    public static void main(String[] args) {
        int j = 0;
        for (int i = 0; i < 100; i++)
              j = j++;
        System.out.println(j);
    }
}
```

上面的程序输出的是 0，而不是 100。

【参考资料】

1.  [java解惑](http://book.douban.com/subject/1473329/)

---EOF---

