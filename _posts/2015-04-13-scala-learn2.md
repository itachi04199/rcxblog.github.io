---
layout: post
title: scala 类和对象
categories: scala
tags: 笔记 scala
---

#### 类

在 scala 当中的类与 java 当中的很类似：

```scala
class ChecksumAccumulator {
	private var sum = 0
    def add(b: Byte): Unit = {
    	sum += b
    }
    def checksum(): Int = {
    	return ~(sum & 0xFF) + 1
    }
}

val acc = new ChecksumAccumulator
val csa = new ChecksumAccumulator
```

public 是 Scala 的缺省访问级别。

如果没有发现任何显式的返回语句，Scala 方法将返回方法中最后一个计算得到的值。所以方法当中的 return 可以省略。

当方法中不返回值的时候，我们可以省略等号，如下代码：

```scala
class ChecksumAccumulator {
	private var sum = 0
	def add(b: Byte) { sum += b }
	def checksum(): Int = ~(sum & 0xFF) + 1
}
```

当去掉方法体前面的等号时，它的结果类型将注定是 Unit。

#### Singleton 对象

在 Scala 当中没有静态成员。Scala 有单例对象：singleton object。除了用 object 关键字替换了 class 关键字以外，单例对象的定义看上去就像是类定义。

```scala
import scala.collection.mutable.Map
object ChecksumAccumulator {
    private val cache = Map[String, Int]()
    def calculate(s: String): Int =
    	if (cache.contains(s))
    		cache(s)
    	else {
            val acc = new ChecksumAccumulator
            for (c <- s)
            	acc.add(c.toByte)
            val cs = acc.checksum()
            cache += (s -> cs)
            cs
    }
}

ChecksumAccumulator.calculate("Every value is an object.")
```

表中的单例对象被叫做 ChecksumAccumulator，与前一个例子里的类同名。当单例对象与某个类共享同一个名称时，他被称作是这个类的伴生对象：companion object。你必须在同一个源文件里定义类和它的伴生对象。类被称为是这个单例对象的伴生类：companion class。类和它的伴生对象可以互相访问其私有成员。

类和单例对象间的一个差别是，单例对象不带参数，而类可以。因为你不能用 new 关键字实例化一个单例对象，你没机会传递给它参数。

单例对象会在第一次被访问的时候初始化。

不与伴生类共享名称的单例对象被称为孤立对象：standalone object。

#### scala 程序

要执行Scala程序，你一定要提供一个有 main 方法（仅带一个参数，Array[String]，且结果类型为 Unit）的孤立单例对象名。

```scala
import ChecksumAccumulator.calculate
object Summer {
    def main(args: Array[String]) {
   		for (arg <- args)
    		println(arg + ": " + calculate(arg))
    }
}
```

Scala 隐式引用了包 java.lang 和 scala 的成员，和名为 Predef 的单例对象的成员，到每个 Scala 源文件中。

可以使用 scalac 命令来编译上面的 scala 文件：

```bash
//编译
$ scalac ChecksumAccumulator.scala Summer.scala
//执行
$ scala Summer of love
//输出
of: -213
love: -182
```

#### Application 特质

可以使用 extends Application 来代替 main 方法：

```scala
import ChecksumAccumulator.calculate
object FallWinterSpringSummer extends Application {
	for (season <- List("fall", "winter", "spring"))
		println(season +": "+ calculate(season))
}
```

你可以把想要放在 main 方法里的代码直接放在单例对象的大括号之间。就这么简单。之后可以像对其它程序那样编译和运行。

【参考资料】

1. [Scala编程](http://book.douban.com/subject/5377415/)

---EOF---

