---
layout: post
title: Java 自动装箱和拆箱
categories: java基础
tags: 笔记
---

装箱就是 java 编译器自动把基本类型转换成对应的封装类型。例如，转换 int 到 Interger，double 到 Double 等等。转换的方向相反的话就是拆箱。

下面是一个装箱的例子：

```java
Character ch = 'a';
```

考虑下面的例子：

```java
List<Integer> li = new ArrayList<>();
for (int i = 1; i < 50; i += 2)
    li.add(i);
```

泛型只能使用基本类型的包装类型，在什么的例子当中，我们把 int 类型的数值传递到了 li 当中，但是编译器不会出错。因为编译器会自动把 int 类型转换成 Interger 类型。编译器编译后的结果相当于下面的代码：

```java
List<Integer> li = new ArrayList<>();
for (int i = 1; i < 50; i += 2)
    li.add(Integer.valueOf(i));
```

在下面两种情况下会自动装箱：

- 调用方法时候，方法参数是包装类型，但是传入基本类型。
- 将基本类型赋值给包装类型。

看下面的例子：

```java
public static int sumEven(List<Integer> li) {
    int sum = 0;
    for (Integer i: li)
        if (i % 2 == 0)
            sum += i;
        return sum;
}
```

由于 % 和 += 操作符不允许操作 Interger 对象，编译器会自动把 Interger 对象转换成 int，所以编译后的代码如下。

```java
public static int sumEven(List<Integer> li) {
    int sum = 0;
    for (Integer i : li)
        if (i.intValue() % 2 == 0)
            sum += i.intValue();
        return sum;
}
```

在下面两种情况下会自动拆箱：

- 调用方法时候，方法参数是基本类型，但是传入包装类型。
- 将包装类型赋值给基本类型。

下面是个拆箱的例子：

```java
import java.util.ArrayList;
import java.util.List;

public class Unboxing {

    public static void main(String[] args) {
        Integer i = new Integer(-8);

        // 1. Unboxing through method invocation
        int absVal = absoluteValue(i);
        System.out.println("absolute value of " + i + " = " + absVal);

        List<Double> ld = new ArrayList<>();
        ld.add(3.1416);    // Π is autoboxed through method invocation.

        // 2. Unboxing through assignment
        double pi = ld.get(0);
        System.out.println("pi = " + pi);
    }

    public static int absoluteValue(int i) {
        return (i < 0) ? -i : i;
    }
}
```

结果会输出为：

```java
absolute value of -8 = 8
pi = 3.1416
```

下面是基本类型和包装类型对应表：

Primitive type|Wrapper class
:--|:--
boolean	|Boolean
byte	|Byte
char	|Character
float	|Float
int|	Integer
long	|Long
short	|Short
double	|Double


【参考资料】

1. [java官方文档](http://docs.oracle.com/javase/tutorial/java/data/autoboxing.html)


---EOF---


