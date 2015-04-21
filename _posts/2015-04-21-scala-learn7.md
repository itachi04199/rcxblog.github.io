---
layout: post
title: scala 函数和闭包下
categories: scala
tags: 笔记 scala
---

### 偏应用函数

可以使用一个下划线替换整个参数列表。

```scala
someNumbers.foreach(println _)
代替
someNumbers.foreach(x => println(x))
```

这个例子中的下划线不是单个参数的占位符。它是整个参数列表的占位符。

以这种方式使用下划线时，你就正在写一个偏应用函数。

```scala
def sum(a: Int, b: Int, c: Int) = a + b + c
sum(1, 2, 3)
val a = sum _
a(1, 2, 3)
```

Scala 编译器以偏应用函数表达式，sum _，实例化一个带三个缺失整数参数的函数值，并把这个新的函数值的索引赋给变量 a。

实际是这样：变量 a 指向一个函数值对象。这个函数值是由 Scala 编译器依照偏应用函数表达式 `sum _`，自动产生的类的一个实例。编译器产生的类有一个 apply 方法带三个参数。

```scala
a.apply(1, 2, 3)
```

还可以进行如下使用：

```scala
val b = sum(1, _: Int, 3)
b(2) // b.apply 调用了 sum(1,2,3)。
```

偏应用函数还可以进行简化：

```scala
someNumbers.foreach(println _)
你可以只是写成：
someNumbers.foreach(println)
```

这种格式仅在需要写函数的地方，如例子中的 foreach 调用，才能使用。

### 闭包

看如下定义：

```scala
(x: Int) => x + more // more 是多少？
```

more 是个自由变量，x 是一个绑定变量。当没定义 more 的时候，编译器会报错。如果定义 more 那么函数文本将工作正常：

```scala
var more = 1
val addMore = (x: Int) => x + more
addMore(10)
```

依照这个函数文本在运行时创建的函数值（对象）被称为闭包。

我们可以创建一个产生闭包的函数：

```scala
def makeIncreaser(more: Int) = (x: Int) => x + more
val inc1 = makeIncreaser(1)
val inc9999 = makeIncreaser(9999)
```

调用 makeIncreaser(1) 时，捕获值 1 当作 more 的绑定的闭包被创建并返回。相似地，调用 makeIncreaser(9999)，捕获值 9999 当作 more 的闭包被返回。

### 重复参数

Scala 允许你指明函数的最后一个参数可以是重复的。想要标注一个重复参数，在参数的类型之后放一个星号。

```scala
def echo(args: String*) = for (arg <- args) println(arg)
```

函数内部，重复参数的类型是声明参数类型的数组。然而，如果你有一个合适类型的数组，并尝试把它当作重复参数传入，你会得到一个编译器错误。要实现这个做法，你需要在数组参数后添加一个冒号和一个_*符号。

```scala
val arr = Array("What's", "up", "doc?")
echo(arr: _*)
```

这个标注告诉编译器把 arr 的每个元素当作参数，而不是当作单一的参数传给 echo。

【参考资料】

1. [Scala编程](http://book.douban.com/subject/5377415/)

---EOF---

