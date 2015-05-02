---
layout: post
title: scala 函数和闭包上
categories: scala
tags: 笔记 scala
---

### 本地函数

在 scala 当中可以在函数中定义另外一个函数，就像本地变量一样，这种本地函数仅在包含它的代码块中可见。

```scala
def processFile(filename: String, width: Int) {
  def processLine(filename:String, width:Int, line:String) {
    if (line.length > width) print(filename+": "+line)
  }
  val source = Source.fromFile(filename)
  for (line <- source.getLines) {
    processLine(filename, width, line)
  }
}
```

本地函数可以访问包含它的函数的参数，所以上面的例子可以修改如下：

```scala
import scala.io.Source
object LongLines {
  def processFile(filename: String, width: Int) {
    def processLine(line: String) {
      if (line.length > width)
        print(filename +": "+ line)
    }
    val source = Source.fromFile(filename)
    for (line <- source.getLines)
      processLine(line)
  }
}
```

### 函数是第一类值

scala 拥有第一类函数，在 scala 当中可以定义函数和调用函数，也可以把函数写成没有名字的文本，并把它们像值一样传递。

函数文本被编译进一个类，类在运行期实例化的时候是一个函数值。函数文本和值的区别在于函数文本存在源代码中，而函数值存在于运行期对象。

看如下例子：

```scala
(x: Int) => x + 1
```

=> 指明这个函数把左边的任何整数 x 转变成右边的 x + 1。所以可以这是一个把 x 映射成 x + 1 的函数。

函数值是对象，可以把它们存入变量。

```scala
var increase = (x: Int) => x + 1
```

如果函数文本超过一行，可以使用大括号：

```scala
var fun = (x: Int) => {
	println("hello world")
    x + 1
}
```

函数文本和函数值在 scala 当中有许多地方可以用到，在集合类中的 foreach 方法和 filter 方法中等：

```scala
val someNumbers = List(-11, -10, -5, 0, 5, 10)
someNumbers.foreach((x: Int) => println(x))
someNumbers.filter((x: Int) => x > 0)
```

### 函数文本短格式

一种让函数文本更简短的方式是去掉参数类型。

```scala
someNumbers.filter((x) => x > 0)
```

编译器知道 x 一定是整数，因为 someNumbers 是包含整型的列表。

第二种简化的方式是省略类型是被推断的参数之外的括号。

```scala
someNumbers.filter(x => x > 0)
```

### 占位符语法

可以使用下划线当做一个或更多参数的占位符，只要每个参数在函数文本内仅出现一次。

```scala
someNumbers.filter(_ > 0)
```

也可以如下使用：

```scala
val f = (_: Int) + (_: Int)
f(5, 10)
```

【参考资料】

1. [Scala编程](http://book.douban.com/subject/5377415/)

---EOF---

