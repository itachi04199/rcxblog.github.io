---
layout: post
title: scala 基础学习
categories: scala
tags: 笔记 scala
---

#### 变量

Scala 有两种变量：

- val，类似于 Java 里的 final 变量，一旦初始化了，val 就不能再赋值了。
- var，如同 Java 里面的非 final 变量，可以在它生命周期中被多次赋值。

```scala
scala> val msg = "Hello, world!"
msg: java.lang.String = Hello, world!
```

本例演示了类型推断，Scala 解释器（或编译器）可以推断类型。

也可以指定类型：

```scala
scala> val msg2: java.lang.String = "Hello again, world!"
msg2: java.lang.String = Hello again, world!

// 在 Scala 程序里 java.lang 类型的简化名也是可见的
scala> val msg3: String = "Hello yet again, world!"
msg3: String = Hello yet again, world!
```

#### 函数

定义一个简单的函数如下：

```scala
def max(x: Int, y: Int): Int = {
	if (x > y)
    	x
    else
    	y
}
```

函数的定义用 def 开始。上面例子的函数名字是 max，括号里面是参数列表，每个函数参数后面必须带前缀冒号的类型标注，因为 Scala 编译器没办法推断函数参数类型。外面的: Int 是函数的结果类型。

函数的结果类型可以省略，但是当函数是递归的情况，就必须显示的定义函数结果类型。同样，如果函数由一个句子组成，可以不写大括号。

```scala
def max2(x: Int, y: Int) = if (x > y) x else y
```

#### 循环

while 循环如下：

```scala
var i = 0
while (i < args.length) {
	println(args(i))
    i += 1
}
```

注意 Java 的 `++i ` 和 `i++`在 Scala 里不起作用，要在 Scala 里自增，必须写成要么 i = i + 1，或者 i += 1。用

函数式的循环：

```scala
args.foreach(arg => println(arg))

args.foreach((arg: String) => println(arg))

args.foreach(println)
```

总而言之，函数文本的语法就是，括号里的命名参数列表，右箭头，然后是函数体。

```scala
  函数参数        右箭头    函数体
(x: Int, y: Int)   =>      x + y
```

#### 数组

创建一个数组如下：

```scala
val greetStrings = new Array[String](3)
greetStrings(0) = "Hello"
greetStrings(1) = ", "
greetStrings(2) = "world!\n"
for (i <- 0 to 2)
	print(greetStrings(i))
```

greetStrings 的类型是 `Array[String]`，不是`Array[String](3)`。

Scala 里的数组是通过把索引放在圆括号里面访问的。

上面的 for 循环当中 0 to 2，这段代码的 to 实际上是带一个 Int 参数的方法。代码 0 to 2 被转换成方法调用 (0).to(2) 。

Scala 没有操作符重载，因为它根本没有传统意义上的操作符。取而代之的是，诸如`+，-，*`和`/`这样的字符可以用来做方法名。

你也可以使用传统的方法调用语法把 1 + 2 替代写成 (1).+(2) 。

Scala 里所有的操作符都是方法调用。

另一重要思想可以让你看到为什么数组在 Scala 里是用括号访问的。

当你在一个或多个值或变量外使用括号时，Scala 会把它转换成对名为 apply的 方法调用。

当然前提是这个类型实际定义过 apply 方法。所以这不是一个特例，而是一个通则。

带有括号里参数和等号右边的对象的 update 方法的调用。

于是 greetStrings(i) 转换成 greetStrings.apply(i)。所以 Scala 里访问数组的元素也只不过是跟其它的一样的方法调用。这个原则不仅仅局限于数组：任何对某些在括号中的参数的对象的应用将都被转换为对 apply 方法的调用。

看下面的转化：

```scala
greetStrings(0) = "Hello"
将被转化为
greetStrings.update(0, "Hello")
```

更简洁的定义数组的方式：

```scala
val numNames = Array("zero", "one", "two")
```

#### List

创建一个 List 很简单:

```scala
val oneTwoThree = List(1, 2, 3)
```

List 有个叫“:::”的方法实现叠加功能。

```scala
val oneTwo = List(1, 2)
val threeFour = List(3, 4)
val oneTwoThreeFour = oneTwo ::: threeFour
println(oneTwo + " and " + threeFour + " were not mutated.")
println("Thus, " + oneTwoThreeFour + " is a new List.")
```

输出结果是：

```scala
List(1, 2) and List(3, 4) were not mutated.
Thus, List(1, 2, 3, 4) is a new List.
```

‘::’ 把一个新元素组合到已有 List 的最前端，然后返回结果 List。

```scala
val twoThree = list(2, 3)
val oneTwoThree = 1 :: twoThree
println(oneTwoThree)
//输出
List(1, 2, 3)
```

下表是 List 的一些方法和作用

方法名|方法作用
:--|:--
List() 或 Nil|空List
List("Cool", "tools", "rule)|创建带有三个值"Cool"，"tools"和"rule"的新List[String]
val thrill = "Will"::"fill"::"until"::Nil|创建带有三个值"Will"，"fill"和"until"的新List[String]
List("a", "b") ::: List("c", "d")|叠加两个列表（返回带"a"，"b"，"c"和"d"的新List[String]）
thrill(2)|返回在thrill列表上索引为2（基于0）的元素（返回"until"）
thrill.count(s => s.length == 4)|计算长度为 4 的 String 元素个数（返回2）
thrill.drop(2)|返回去掉前2个元素的thrill列表（返回List("until")）
thrill.dropRight(2)|返回去掉后2个元素的thrill列表（返回List("Will")）
thrill.exists(s => s == "until")|判断是否有值为"until"的字串元素在thrill里（返回true）
thrill.filter(s => s.length == 4)|依次返回所有长度为4的元素组成的列表（返回List("Will", "fill")）
thrill.forall(s => s.endsWith("1"))|辨别是否thrill列表里所有元素都以"l"结尾（返回true）
thrill.foreach(s => print(s))|对thrill列表每个字串执行print语句（"Willfilluntil"）
thrill.foreach(print)|与前相同，不过更简洁（同上）
thrill.head|返回thrill列表的第一个元素（返回"Will"）
thrill.init|返回thrill列表除最后一个以外其他元素组成的列表（返回List("Will", "fill")）
thrill.isEmpty|说明thrill列表是否为空（返回false）
thrill.last|返回thrill列表的最后一个元素（返回"until"）
thrill.length|返回thrill列表的元素数量（返回3）
thrill.map(s => s + "y")|返回由thrill列表里每一个String元素都加了"y"构成的列表（返回List("Willy", "filly", "untily")）
thrill.mkString(", ")|用列表的元素创建字串（返回"will, fill, until"）
thrill.remove(s => s.length == 4)|返回去除了thrill列表中长度为4的元素后依次排列的元素列表（返回List("until")）
thrill.reverse|返回含有thrill列表的逆序元素的列表（返回List("until", "fill", "Will")）
thrill.sort((s, t) => s.charAt(0).toLowerCase < t.charAt(0).toLowerCase)|返回包括thrill列表所有元素，并且第一个字符小写按照字母顺序排列的列表（返回List("fill", "until", "Will")）
thrill.tail|返回除掉第一个元素的thrill列表（返回List("fill", "until")）

#### Tuple

另一种有用的容器对象是元组：tuple。与列表一样，元组也是不可变的，但与列表不同，元组可以包含不同类型的元素。

```scala
val pair = (99, "Luftballons")
println(pair._1)
println(pair._2)
```

Scala 推断元组类型为 Tuple2[Int, String]，并把它赋给变量 pair。

#### Set 和 Map

Set 提供了两种实现，一种是可变类型的，一种是不可变的。默认提供的是不可变类型的。

```scala
var jetSet = Set("Boeing", "Airbus")
jetSet += "Lear"
println(jetSet.contains("Cessna"))
```

如果你需要可变集，就需要使用一个引用：import

```scala
import scala.collection.mutable.Set
val movieSet = Set("Hitch", "Poltergeist")
movieSet += "Shrek"
println(movieSet)
```

Map 类型也有两种 scala.collection 包里面有一个基础 Map 特质和两个子特质 Map：可变的 Map 在 scala.collection.mutable 里，不可变的在 scala.collection.immutable 里。同时不可变也是缺省的。

```scala
import scala.collection.mutable.Map
val treasureMap = Map[Int, String]()
treasureMap += (1 -> "Go to island.")
treasureMap += (2 -> "Find big X on ground.")
treasureMap += (3 -> "Dig.")

println(treasureMap(2))
```

更简单的创建如下：

```scala
val romanNumeral = Map(
	1 -> "I", 2 -> "II", 3 -> "III", 4 -> "IV", 5 -> "V"
)
println(romanNumeral(4))
```

【参考资料】

1. [Scala编程](http://book.douban.com/subject/5377415/)

---EOF---

