---
layout: post
title: scala 基本类型和操作
categories: scala
tags: 笔记 scala
---

### 基本类型

类型 Byte，Short，Int，Long和 Char 被称为整数类型。整数类型加上 Float 和 Double 被称为数类型。

除了 String 归于 java.lang 包之外，其余所有的基本类型都是包 scala 的成员。如 scala.Int。

并且 每个源文件都会自动引用 scala 和 java.lang 的所有成员。

### 文本

文本是直接在代码里写常量值的一种方式。

```scala
val hex = 0x5
hex: Int = 5

val hex2 = 0x00FF
hex2: Int = 255

val nov = 0777
nov: Int = 511

val tower = 35L
tower: Long = 35

val big = 1.2345
big: Double = 1.2345

val little = 1.2345F
little: Float = 1.2345

val a = 'A'
a: Char = A

val c = '\101'
c: Char = A

val f = '\u0044'
f: Char = D

val backslash = '\\'
backslash: Char = \

val hello = "hello"
hello: java.lang.String = hello

val bool = true
bool: Boolean = true
```

在 Scala 中以三个引号（"""）开始和结束一条原始字串。内部的原始字串可以包含无论何种任意字符，包括新行，引号和特殊字符，当然同一行的三个引号除外。

```scala
println("""Welcome to Ultamix 3000. Type "HELP" for help.""")

Welcome to Ultamix 3000. Type "HELP" for help.
```

### 符号文本

符号文本被写成 `'<标识符>`，这里 `<标识符>` 可以是任何字母或数字的标识符。这种文本被映射成预定义类 scala.Symbol 的实例。

```scala
scala> val s = 'aSymbol
s: Symbol = 'aSymbol
```

### 操作符和方法

```scala
scala> val sum = 1 + 2 // Scala调用了(1).+(2)
sum: Int = 3
```

Int 包含了许多带不同的参数类型的重载方法。还有另一个也叫 + 的方法参数和返回类型为 Long。如果你把 Long 加到 Int 上，这个替换的 + 方法就将被调用：

```scala
scala> val longSum = 1 + 2L // Scala调用了(1).+(2L) longSum: Long = 3
```

符号 + 是操作符，是中缀操作符。你可以把任何方法都当作操作符来标注。

```scala
scala> val s = "Hello, world!"
s: java.lang.String = Hello, world!
scala> s indexOf 'o' // Scala调用了s.indexOf(’o’)
res0: Int = 4
scala> s indexOf ('o', 5) // Scala调用了s.indexOf(’o’, 5)
res1: Int = 8
```

Scala 还有另外两种操作符标注：前缀和后缀。

```scala
-7 // 前缀
7 tolang // 后缀
```

Scala 会把表达式 -2.0 转换成方法调用 “(2.0).unary_-”。

```scala
scala> -2.0 // Scala调用了(2.0).unary_-
res2: Double = -2.0
scala> (2.0).unary_-
res3: Double = -2.0
```

前缀操作符用的标识符只有 +，-，! 和 ~。如果你定义了名为 unary_! 的方法，就可以像 !p 这样在合适的类型值或变量上用前缀操作符方式调用这个方法。

但是定义了名为 `unary_*` 的方法，就不是前缀操作符，因为 `*` 不是前缀操作符之一。你可以像平常那用调用它，如 `p.unary_*`，但如果尝试像 `*p` 这么调用，Scala 就会把它理解为 `*.p`。

后缀操作符是不用点或括号调用不带任何参数的方法。

如果方法带有副作用就加上括号，如 println()，不过如果方法没有副作用就可以去掉括号，如 String 上调用的 toLowerCase：

```scala
scala> val s = "Hello, world!"
s: java.lang.String = Hello, world!
scala> s toLowerCase
res5: java.lang.String = hello, world!
```

### 数学运算

```scala
scala> 1.2 + 2.3
res6: Double = 3.5
scala> 3 - 1
res7: Int = 2
scala> 'b' - 'a'
res8: Int = 1 scala> 2L * 3L
res9: Long = 6
scala> 11 / 4
res10: Int = 2
scala> 11 % 4
```

### 关系和逻辑操作

```scala
scala> 1 > 2
res16: Boolean = false
scala> 1 < 2
res17: Boolean = true
scala> 1.0 <= 1.0
res18: Boolean = true
scala> 3.5f >= 3.6f
res19: Boolean = false
scala> 'a' >= 'A'
res20: Boolean = true

scala> val toBe = true
toBe: Boolean = true
scala> val question = toBe || !toBe
question: Boolean = true
scala> val paradox = toBe && !toBe
paradox: Boolean = false
```

与 Java 里一样，逻辑与和逻辑或有短路。

### 位运算

```scala
scala> 1 & 2
res24: Int = 0
scala> 1 | 2
res25: Int = 3
scala> 1 ˆ 3
res26: Int = 2
scala> ~1
res27: Int = -2
```

### 对象相等性

```scala
scala> 1 == 2
res24: Boolean = false
scala> 1 != 2
res25: Boolean = true
scala> List(1, 2, 3) == List(1, 2, 3)
res27: Boolean = true
scala> List(1, 2, 3) == List(4, 5, 6)
res28: Boolean = false
scala> 1 == 1.0
res29: Boolean = true
scala> List(1, 2, 3) == "hello"
res30: Boolean = false
scala> List(1, 2, 3) == null
res31: Boolean = false
scala> null == List(1, 2, 3)
res32: Boolean = false
scala> ("he" + "llo") == "hello"
res33: Boolean = true
```

首先检查左侧是否为 null，如果不是，调用 equals 方法。由于 equals 是一个方法，因此比较的精度取决于左手边的参数。

### 操作符优先级

由于 Scala 没有操作符，实际上，是以操作符的格式使用方法的一个途径，Scala 基于操作符格式里方法的第一个字符决定优先级，如果方法名开始于 `*`,那么就比开始于 `+` 的方法有更高的优先级。

操作符优先级由高到低如下：

```scala
* / %
+ -
:
= !
< >
&
^
|
（所有字母）
```

有关于以等号结束的赋值操作符，它比任何其他操作符的优先级都低。

```scala
x *= y + 1
与下面的相同：
x *= (y + 1)
```

操作符的关联性决定了操作符分组的方式。Scala 里操作符的关联性取决于它的最后一个字符。任何以 ‘:’ 字符结尾的方法由它的右手侧操作数调用，并传入左操作数。以其他字符结尾的方法有其他的说法。它们都是被左操作数调用，并传入右操作数。

```scala
a * b 变成 a.*(b)
a:::b 变成 b.:::(a)
```

不论操作符具有什么样的关联性，它的操作数总是从左到右评估的。

a:::b 将会当作是：

```scaka
{ val x = a; b.:::(x) }
```
这个代码块中，a 仍然在 b 之前被评估，然后评估结果被当作操作数传给 b 的 ::: 方法。

### 富包装器

基本类型|富包装
:--|:--
Byte|scala.runtime.RichByte
Short|scala.runtime.RichShort
Int|scala.runtime.RichInt
Long|scala.runtime.RichLong
Char|scala.runtime.RichChar
String|scala.runtime.RichString
Float|scala.runtime.RichFloat
Double|scala.runtime.RichDouble
Boolean|scala.runtime.RichBoolean

【参考资料】

1. [Scala编程](http://book.douban.com/subject/5377415/)

---EOF---

