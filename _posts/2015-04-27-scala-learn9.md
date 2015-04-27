---
layout: post
title: scala 控制抽象
categories: scala
tags: 笔记 scala
---

### 减少代码重复

我们可以使用高阶函数来简化代码重复，高阶函数是参数中包含函数的函数。看下面的例子：

```scala
object FileMatcher {
  private def filesHere = (new java.io.File(".")).listFiles
  def filesEnding(query: String) =
    for (file <- filesHere; if file.getName.endsWith(query))
      yield file
}
```

上面的例子是查询当前目录下文件名字跟指定的名字相同的文件，如果我们要扩展的话还可能加上 contains 或者 endsWith 或者正则表达式匹配的过滤条件。这样就如下代码了，重复代码非常的多：

```scala
object FileMatcher {
  private def filesHere = (new java.io.File(".")).listFiles
  def filesEnding(query: String) =
    for (file <- filesHere; if file.getName.endsWith(query))
      yield file

  def filesEnding(query: String) =
    for (file <- filesHere; if file.getName.endsWith(query))
      yield file

  def filesContaining(query: String) =
  	for (file <- filesHere; if file.getName.contains(query))
		yield file

  def filesRegex(query: String) =
  	for (file <- filesHere; if file.getName.matches(query))
		yield file
}
```

在上面的例子当中，我们可以抽取出来一个高级函数，参数当中传入一个函数来当做匹配条件：

```scala
object FileMatcher {
      private def filesHere = (new java.io.File(".")).listFiles
    def filesMatching(query: String,
        matcher: (String, String) => Boolean) = {
      for (file <- filesHere; if matcher(file.getName, query))
        yield file
    }
    def filesEnding(query: String) =
      filesMatching(query, _.endsWith(_))
    def filesContaining(query: String) =
      filesMatching(query, _.contains(_))
    def filesRegex(query: String) =
  	  filesMatching(query, _.matches(_))
}
```

当然 query 参数我们也可以不用传递来传递去的，可以省略传递：

```scala
object FileMatcher {
  private def filesHere = (new java.io.File(".")).listFiles
  private def filesMatching(matcher: String => Boolean) =
    for (file <- filesHere; if matcher(file.getName))
      yield file
  def filesEnding(query: String) =
    filesMatching(_.endsWith(query))
  def filesContaining(query: String) =
    filesMatching(_.contains(query))
  def filesRegex(query: String) =
    filesMatching(_.matches(query))
}
```

scala 提供了很多高阶函数来简化我们客户端代码，例如在一个集合当中判断是否存在符合某种条件的元素。

```scala
def containsNeg(nums: List[Int]) = nums.exists(_ < 0)
```

### curry 化

我们定义个最简单的函数：

```scala
def plainOldSum(x: Int, y: Int) = x + y
```

如果我们要对上面的函数进行 curry 化，就成为如下：

```scala
def plainOldSum(x: Int)(y: Int) = x + y
```

当我们 curry 化之后，在调用 plainOldSum 函数的时候就相当于调用两个传统函数，第一个函数的形式如下：

```scala
def first(x: Int) = (y: Int) => x + y
```

当在第一个函数上应用 1，那么第二个函数会如下:

```scala
val second = first(1)
```

我们如果要使用 curry 化的函数是需要指定第二个参数列表的占位符。

```scala
val onePlus = plainOldSum(1)_
```

### 编写新的控制结构

看如下一个函数的例子：

```scala
def twice(op: Double => Double, x: Double) = op(op(x))
twice(_ + 1, 5)
```

在上面的例子当中 op 的类型是 Double => Double，就是它带一个 Double 参数并且返回一个 Double 的函数。

当发现代码当中包含重复的控制模式的时候，我们就应该考虑把它实现为一个新的控制结构。一个更加广泛的模式：打开资源，操作资源，关闭资源。可以使用如下方法：

```scala
def withPrintWriter(file: File, op: PrintWriter => Unit) {
  val writer = new PrintWriter(file)
  try {
    op(writer)
  } finally {
    writer.close()
  }
}
// 可以如下使用：
withPrintWriter(
  new File("date.txt"),
  writer => writer.println(new java.util.Date)
)
```

这个函数只传入了需要处理的文件和，对文件如何进行处理，而不需要担心对文件的关闭忘记引起问题。

也可以使用大括号来代替小括号包围参数列表。scala 的任何方法调用，如果传入一个参数，就可以使用大括号代替小括号包围参数。

```scala
println("Hello, world!")
println{"Hello, world!"}
```

在 withPrintWriter 方法就不能使用大括号，因为这个方法包含了两个参数，但是我们可以使用 curry 化把第一个参数分离。

```scala
def withPrintWriter(file: File)(op: PrintWriter => Unit) {
  val writer = new PrintWriter(file)
  try {
    op(writer)
  } finally {
    writer.close()
  }
}
// 如下使用
val file = new File("date.txt")
withPrintWriter(file) {
  writer => writer.println(new java.util.Date)
}
```

### 叫名参数

在上面的 withPrintWriter 例子当中，我们需要传递一个 PrintWriter 参数，这个参数使用 writer => 方式表现，如果想实现更像 if 或 while 的东西，没有值要传入大括号之间的代码，我们可以使用叫名参数。

实现一个 myAssert 的断言例子，myAssert 函数将带一个函数值做输入并参考一个标志位来决定该做什么。如果标志位被设置了 myAssert 将调用传入的函数并证实其返回 true。如果标志位被关闭了,myAssert 将安静地什么都不做。

```scala
var assertionsEnabled = true
def myAssert(predicate: () => Boolean) =
  if (assertionsEnabled && !predicate())
    throw new AssertionError
// 使用
myAssert(() => 5 > 3)
// 我们想写成如下
myAssert(5 > 3)
```

当要实现一个叫名函数，在定义参数类型开始于 => 而不是 ()=>。所以上面的例子可以改成如下：

```scala
def byNameAssert(predicate: => Boolean) =
  if (assertionsEnabled && !predicate)
	￼throw new AssertionError
```

叫名类型中,空的参数列表,(),被省略,它仅在参数中被允许。

当前我们也可以进行如下的简化：

```scala

def boolAssert(predicate: Boolean) =
  if (assertionsEnabled && !predicate)
	throw new AssertionError
```

boolAssert 函数的参数是一个布尔值。看上去使用这两个函数的方式是一样的，但是他们之间存在一个非常重要的差别。在 boolAssert(5 > 3) 括号中的表达式先于 boolAssert 的调用被评估，byNameAssert(5 > 3) 里括号中的表达式不是先于byNameAssert的调用被评估的。

上面的差别会导致：

```scala
boolAssert(x / 0 == 0) // 会产生异常
byNameAssert(x / 0 == 0) // 不会产生异常
```

【参考资料】

1. [Scala编程](http://book.douban.com/subject/5377415/)

---EOF---

