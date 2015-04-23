---
layout: post
title: scala 包和引用
categories: scala
tags: 笔记 scala
---

### 包

scala 当中的包跟 java 当中的包类似：

```scala
package com.rcx
```

还有一种情况，包可以进行嵌套：

```scala
package bobsrockets {
    package navigation {  // 在bobsrockets.navigation包中
    	class Navigator
    	package tests { // 在bobsrockets.navigation.tests包中
    		class NavigatorSuite
    	}
    }
}
```

下边是一个使用嵌套包当中的类的例子：

```scala
// 文件launch.scala
package launch {
	class Booster3
}

// 文件bobsrockets.scala
package bobsrockets {
    package navigation {
        package launch {
        	class Booster1
        }
        class MissionControl {
            val booster1 = new launch.Booster1
            val booster2 = new bobsrockets.launch.Booster2
            val booster3 = new _root_.launch.Booster3
        }
    }
    package launch {
    	class Booster2
    }
}
```

Scala 提供了所有用户可创建的包之外的名为 `_root_` 的包。换句话就是，任何你写的顶层包都被当作是 `_root_` 包的成员。

### 引用

可以通过 import 来引入其他包当中的类：

```scala
import bobsdelights.Fruit // 易于访问 Fruit
import bobsdelights._ // 易于访问 bobsdelights 的所有成员
import bobsdelights.Fruits._ // 易于访问 Fruits 的所有成员
```

与 java 当中引用的一个区别是，在 java 当中引用所有尾部是 * 而不是 _。

Scala 引用可以出现在任何地方，而不是仅仅在编译单元的开始处。

```scala
def showFruit(fruit: Fruit) {
	import fruit._
    println(name +"s are "+ color)
}
```

方法 showFruit 引用了它的参数，Fruit 类型的 fruit 的所有成员。这两个索引等价于 fruit.name 和 fruit.color。

还有一些特殊的引入方式：

```scala
import Fruits.{Apple, Orange} // 引用了对象 Fruits 的 Apple 和 Orange 成员。
import Fruits.{Apple => McIntosh, Orange} // 引用 Apple 和 Orange 但是 Apple 重命名为 McIntosh。
import java.{sql => S} // 以名称 S 引用了 java.sql 包，这样你就可以写成 S.Date。
import Fruits.{_} // 与import Fruits._同义。
import Fruits.{Pear => _, _} // 引用了除 Pear 之外的所有 Fruits 成员。
```

Scala隐式地添加了一些引用到每个程序中，如下这三个引用。

```acala
import java.lang._ // java.lang 包的所有东西
import scala._ // scala 包的所有东西
import Predef._ // Predef 对象的所有东西
```

### 访问修饰符

**私有成员：**

```scala
class Outer {
    class Inner {
        private def f() {
       		println("f")
        }
        class InnerMost {
        	f() // OK
        }
    }
    (new Inner).f() // 错误：f不可访问
}
```

Scala 里，(new Inner).f() 访问非法，因为 f 在 Inner 中被声明为 private 而访问不在类 Inner 之内。相反，类 InnerMost 里访问 f 没有问题，因为这个访问包含在 Inner 类之内。Java 会允许这两种访问因为它允许外部类访问其内部类的私有成员。

**保护成员：**

Scala 里，保护成员只在定义了成员的类的子类中可以被访问。Java 中，这种访问同样可以在类的同一个包里。

```scala
package p {
class Super {
	protected def f() { println("f") }
}
class Sub extends Super {
	f()
}
class Other {
	(new Super).f() // 错误：f不可访问
}
}
```

**公开成员：**

任何没有标记为 private 或 protected 的是公开的。公开成员没有显式修饰符。

Scala 里的访问修饰符可以通过使用修饰词增加。

看下面的例子来看更加细粒度的设置访问权限：

```scala
package bobsrockets {
	package navigation {
    	private[bobsrockets] class Navigator {
        	protected[navigation] def useStarChart() {}
        class LegOfJourney {
        	private[Navigator] val distance = 100
        }
        private[this] var speed = 200
     }
     package launch {
         import navigation._
         object Vehicle {
         	private[launch] val guide = new Navigator
         }
     }
 }
```

类 Navigator 被标记为 private[bobsrockets]。这就是说这个类对包含在 bobsrockets 包的所有的类和对象可见。

LegOfJourney 里的 distance 变量被标记为 private[Navigator]，因此它在类 Navigator 的任何地方都可见。

private[this] 标记的定义仅能在包含了定义的同一个对象中被访问。

类 Navigator 的的 speed 定义就是对象私有的，因此在 Navigator内访问 “speed” 和 this.speed 是合法的。下面的访问是不可以的：

```scala
val other = new Navigator
other.speed // this line would not compile
```

### 可见度和伴生对象

```scala
class Rocket {
    import Rocket.fuel
    private def canGoHomeAgain = fuel > 20
}
object Rocket {
    private def fuel = 10
    def chooseStrategy(rocket: Rocket) {
        if (rocket.canGoHomeAgain)
            goHome()
        else
            pickAStar()
    }
    def goHome() {}
    def pickAStar() {}
}
```

Scala 的访问规则给予了伴生对象和类一些特权。类把它所有的访问权限共享给半生对象，反过来也是如此。上面的 Rocket 类可以访问方法 fuel，它在 Rocket 对象中被声明为私有，Rocket 对象也可以访问 Rocket 类里面的私有方法 canGetHome。

【参考资料】

1. [Scala编程](http://book.douban.com/subject/5377415/)

---EOF---

