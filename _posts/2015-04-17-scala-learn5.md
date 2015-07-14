---
layout: post
title: scala 函数式对象
categories: scala
tags: 笔记 scala
---

通过一个 Rational 的例子来演示函数式对象，n 是分子，d 是分母。

### 类参数和主构造器

函数式对象的一个最大的特点就是内部的状态不可以改变。

定义带参数的类的时候可以如下：

```scala
class Rational(n: Int, d: Int)
```

n 和 d 是类参数，编译器会收集这两个参数并创造一个带同样的两个参数的主构造器。

scala 编译器也会把在类内部的任何不是字段的部分或者方法，编译到主构造器中。

```scala
class Rational(n: Int, d: Int) {
	println("Created "+n+"/"+d)
}
```

Scala 编译器将把 println调用放在 Rational 的主构造器。

### toString 方法

可以重载 toString 方法：

```scala
class Rational(n: Int, d: Int) {
	override def toString = n +"/"+ d
}
```

### 检查先决条件

由于分母不能为零，所以当我们创建对象的时候需要对分母进行检查。

这种情况可以使用 require 方法：

```scala
class Rational(n: Int, d: Int) {
	require(d != 0)
	override def toString = n +"/"+ d
}
```

require 方法带一个布尔型参数。如果传入的值为真，require 将正常返回。反之，require 将通过抛出 IllegalArgumentException 来阻止对象被构造。

### 字段

当我们想要添加一个 add 方法的时候我们可能会如下写法：

```scala
class Rational(n: Int, d: Int) {
	require(d != 0)
	override def toString = n +"/"+ d
    def add(that: Rational): Rational = new Rational(n * that.d + that.n * d, d * that.d)
}
```

但是当编译的时候会报错，是因为通过 that.n 和 that.d 访问的时候是不可行的，我们需要把 n 和 d 放到字段当中。

```scala
class Rational(n: Int, d: Int) {
	require(d != 0)
    val numer: Int = n
    val denom: Int = d
	override def toString = numer +"/"+ denom
    def add(that: Rational): Rational = new Rational(numer * that.denom + that.numer * denom, denom* that.denom)
}
```

### 从构造器

类可以有多个构造器。Scala 里主构造器之外的构造器被称为从构造器。例如：Rational(5) 其实就是 Rational(5,1)，我们想在构造的时候省略后面的参数，并且从构造器预先设置分母为 1。

```scala
class Rational(n: Int, d: Int) {
	require(d != 0)
    val numer: Int = n
    val denom: Int = d
    def this(n: Int) = this(n, 1)
	override def toString = numer +"/"+ denom
    def add(that: Rational): Rational = new Rational(numer * that.denom + that.numer * denom, denom* that.denom)
}
```

Scala 的从构造器开始于 def this(...)。

### 私有字段和方法

例如分数 66/42 ，可以更约简化为相同的最简形式，11/7，所以需要在 Rational 类内部存放一个私有的字段为分子和分母的最大公约数。

```scala
class Rational(n: Int, d: Int) {
	require(d != 0)
    private val g = gcd(n.abs, d.abs)
    val numer: Int = n / g
    val denom: Int = d / g
    def this(n: Int) = this(n, 1)
	override def toString = numer +"/"+ denom
    def add(that: Rational): Rational = new Rational(numer * that.denom + that.numer * denom, denom* that.denom)
    private def gcd(a: Int, b: Int): Int = if (b==0) a else gcd(b, a%b)
}
```

### 定义操作符

整型和浮点可以使用 x + y，如果我们想把 Rational 改成可以使用 + 的话，只需要把 add 方法的名字换成 +，因为我们可以用 + 来定义方法名。

```scala
class Rational(n: Int, d: Int) {
	require(d != 0)
    private val g = gcd(n.abs, d.abs)
    val numer: Int = n / g
    val denom: Int = d / g
    def this(n: Int) = this(n, 1)
	override def toString = numer +"/"+ denom
    def +(that: Rational): Rational = new Rational(numer * that.denom + that.numer * denom, denom* that.denom)
    private def gcd(a: Int, b: Int): Int = if (b==0) a else gcd(b, a%b)
}
```

### Scala 的标识符

Scala 里两种构成标识符的方式：字母数字式和操作符。

字母数字标识符：起始于一个字母或下划线，之后可以跟字母，数字，或下划线。‘$’ 字符也被当作是字母，一般不建议使用。

操作符标识符：由一个或多个操作符字符组成，如+，:，?，~或#。

混合标识符：由字母数字组成，后面跟着下划线和一个操作符标识符。unary_+ 被用做定义一元的 ‘+’ 操作符的方法名。

### 方法重载

当前的 Rational 只能两个 Rational 进行相加，如果要实现 Rational 和一个 Int 可以进行相加，我们需要重载 + 方法。

```scala
class Rational(n: Int, d: Int) {
	require(d != 0)
    private val g = gcd(n.abs, d.abs)
    val numer: Int = n / g
    val denom: Int = d / g
    def this(n: Int) = this(n, 1)
	override def toString = numer +"/"+ denom
    def +(that: Rational): Rational = new Rational(numer * that.denom + that.numer * denom, denom* that.denom)
    def +(i: Int): Rational = new Rational(numer + i * denom, denom)
    private def gcd(a: Int, b: Int): Int = if (b==0) a else gcd(b, a%b)
}
```

### 隐式转换

我们可以写 r + 2，但是不可以写成 2 + r，这是因为 2.+(r)，如果我们想把 Int 隐式转换到 Rational，可以在解释器中加入如下：

```scala
implicit def intToRational(x: Int) = new Rational(x)
```

【参考资料】

1. [Scala编程](http://book.douban.com/subject/5377415/)

---EOF---


