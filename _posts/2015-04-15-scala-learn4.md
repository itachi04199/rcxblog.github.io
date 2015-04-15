---
layout: post
title: scala 内建控制结构
categories: scala
tags: 笔记 scala
---

### if

看如下一个例子：

```scala
var filename = "default.txt"
if (!args.isEmpty)
	filename = args(0)
```

由于 if 表达式是可以返回值的，所以上面的例子可以改写成如下：

```scala
val filename = if (!args.isEmpty) args(0) else "default.txt"
```

### while 循环

scala 当中的 while 循环与其他语言中的一样。

```scala
while (a != 0) {
	val temp = a
    a = b % a
    b = temp
}

do {
	line = readLine()
    println("Read: " + line)
} while (line != null)
```

while 和 do-while 是循环，不是表达式。他们不产生结果，结果的类型是 Unit。Unit 类型的值只有一个是 ()。

```scala
scala> def greet() { println("hi") }
greet: ()Unit
scala> greet() == ()
hi
res0: Boolean = true
```

由于 while 循环不产生值，它经常被纯函数式语言所舍弃。

下面是一个使用递归实现最大公约数的函数：

```scala
def gcd(x: Long, y: Long): Long = if (y == 0) x else gcd(y, x % y)
```

### for

**使用 for 循环遍历枚举集合类**

```scala
val files = (new java.io.File(".")).listFiles
for (file <- files)
	println(file)

for (i <- 1 to 4)
	println("Iteration " + i)

for (i <- 1 until 4)
	println("Iteration " + i)
```

在 for 循环当中进行过滤

```scala
val files = (new java.io.File(".")).listFiles
for (file <- files if file.getName.endsWith(".scala"))
	println(file)
```

在 java 当中的写法是如下：

```scala
for (file <- files)
	if (file.getName.endsWith(".scala"))
    	println(file)
```

同时可以在 for 循环当中添加多个过滤器：

```scala
for (
	file <- filesHere
    if file.isFile;//如果在发生器中加入超过一个过滤器，if子句必须用分号分隔。
    if file.getName.endsWith(".scala")
) println(file)
```

**嵌套枚举**

如果加入多个 <- 子句，你就得到了嵌套的“循环”。

看下面例子：

```scala
def fileLines(file: java.io.File) = scala.io.Source.fromFile(file).getLines.toList
def grep(pattern: String) =
	for {
        file <- filesHere
        if file.getName.endsWith(".scala")
            line <- fileLines(file)
            if line.trim.matches(pattern)
        } println(file + ": " + line.trim)
    grep(".*gcd.*")
```

使用 for 的时候可以使用大括号代替小括号，这样可以省略多个 if 后面的分号。

上面的例子当中 line.trim 重复出现了 2 次，我们可以在 for 当中绑定一个变量，但是不用带关键字 val

```scala
def fileLines(file: java.io.File) = scala.io.Source.fromFile(file).getLines.toList
def grep(pattern: String) =
	for {
        file <- filesHere
        if file.getName.endsWith(".scala")
            line <- fileLines(file)
            trimmed = line.trim
            if trimmed.matches(pattern)
        } println(file + ": " + trimmed)
    grep(".*gcd.*")
```

### try

抛出异常与 java 当中的形式一样：

```scala
throw new IllegalArgumentException
```

捕获异常的形式如下：

```scala
import java.io.FileReader
import java.io.FileNotFoundException
import java.io.IOException

try {
	val f = new FileReader("input.txt")
} catch {
	case ex: FileNotFoundException => // Handle missing file
	case ex: IOException => // Handle other I/O error
}
```

finally 子句

```scala
import java.io.FileReader
val file = openFile()
try {
	// 使用文件
} finally {
	file.close() // 确保关闭文件
}
```

try-catch-finally 也产生值，如果没有异常抛出，则对应于 try 子句；如果抛出异常并被捕获，则对应于相应的 catch 子句。如果异常被抛出但没被捕获，表达式就没有返回值。由 finally 子句计算得到的值，如果有的话，被抛弃。

### match 表达式

看下简单的例子：

```scala
val firstArg = if (args.length > 0) args(0) else ""
firstArg match {
    case "salt" => println("pepper")
    case "chips" => println("salsa")
    case "eggs" => println("bacon")
    case _ => println("huh?")
}
```

match 表达式与 java 当中的 switch 有几点区别，scala 当中的 case 可以匹配任何种类的常量。并且每个选项后面不需要 break，默认包含了 break。

match 表达式也是产生值的，如下：

```scala
val firstArg = if (!args.isEmpty) args(0) else ""
val friend =
	firstArg match {
        case "salt" => "pepper"
        case "chips" => "salsa"
        case "eggs" => "bacon"
        case _ => "huh?"
    }
println(friend)
```

### break 和 continue

在 java 当中 break 和 continue 会经常用在循环当中，但是在 scala 当中一般不建议使用循环，一般都会使用递归函数重写循环。

```scala
var i = 0
var foundIt = false
while (i < args.length && !foundIt) {
    if (!args(i).startsWith(""))
    {
        if (args(i).endsWith(".scala"))
        	foundIt = true
    }
	i = i + 1
}
```

上面的写法是类似在 java 当中的写法，但是可以改写成如下这样：

```scala
def searchFrom(i: Int): Int =
	if (i >= args.length) -1// 不要越过最后一个参数
    else if (args(i).startsWith("-")) searchFrom(i + 1)// 跳过选项
    else if (args(i).endsWith(".scala")) i // 找到！
    else searchFrom(i + 1) // 继续找
val i = searchFrom(0)
```

### 变量范围

scala 的变量范围与 java 当中的基本类似。只有一个差别，scala 允许在嵌套范围内定义同名变量。

```scala
val a = 1; {
	val a = 2 // 编译通过
	println(a)
}
println(a)
```

【参考资料】

1. [Scala编程](http://book.douban.com/subject/5377415/)

---EOF---

