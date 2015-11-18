---
layout: post
title: Java8 lambda 表达式初体验上
categories: java基础
tags: 笔记, java
---

lambda 表达式，是一段可以传递的代码，可以被多次执行。在 java8 之前，如果我们想写一个简单的比较器 Compartor ，我们需要创建一个实现类或者一个匿名内部类类传入到需要比较的方法内当中。

在 java8 之前传递一段代码不是很容易，现在我们想要实现一个通过传递代码来检查某个字符串的长度是否小于另外一个字符串的长度。

```java
(String first, String second) -> Integer.compare(first.length(), second.length());
```

上面这段代码就是 lambda 表达式，这个表达式不仅仅是一个简单的代码块，还必须指定传递给代码的所有变量。

Java 当中 lambda 表达式的格式是：参数、箭头(->)、以及一个表达式。如果负责计算的代码无法用一个表达式表示，可以使用 {} 括起来。

如果 lambda 没有参数，可以使用 () 来表示，如果 lambda 表达式的参数类型可以被推导，那么可以省略掉。

```java
Comparator<String> comparator = (first, second) -> Integer.compare(
				first.length(), second.length());
```

上面的例子当中会推导出 first 和 second 的类型是 String ，因为表达式赋值给了一个字符串比较器。

**注意**，在 lambda 表达式当中只在某些分支有返回值是不合法的。

## 函数式接口

Java 当中有许多接口都需要封装代码块， Runnable 、 Compartor  等等。

对于只包含一个方法的接口，可以通过 lambda 表达式来创建该接口的对象，这种接口被称为函数式接口。

在 java.util.function 包下面提供了许多通用的函数式接口。

可以在任意函数式接口上面使用 @FunctionalInterface 来标识它是一个函数式接口，但是该注解不是强制的。

当 lambda 表达式被转换成一个函数式接口的实例时，需要注意处理检查时异常，如下代码。

```java
Runnable runnable = () -> {
			try {
				Thread.sleep(1000);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		};
```

如果不加 try catch 语句的话，这个赋值语句就会编译错误，因为 Runnable 的 run 方法是没有异常抛出的。

Callable 是可以抛出任何异常，并且有返回值，但是我们不想返回任何数据的时候可以如下定义：

```java
Callable<Void> callable = () -> {
			System.out.println("xxx");
			return null;
		};
```

## 方法引用

有时候，我们想传递的代码已经有现成的实现了。例如，我们仅仅想点击按钮时候打印 event 对象，可以进行如下代码：

```java
button.setOnAction(System.out::println);
```

表达式 System.out::println 是一个方法引用，等同于 lambda 表达式 x -> System.out.println(x);

:: 操作符将方法名和对象或类的名字分隔开。有以下三种主要的使用情况：

- 对象 :: 实例方法
- 类 :: 静态方法
- 类 :: 实例方法

前两种情况，方法引用相当于提供方法参数的 lambda 表达式。 System.out::println 等同于 x -> System.out.println(x);

Math::pow 等同于 (x, y) -> Math.pow(x, y);

第三种情况，第一个参数会成为执行方法的对象。例如， Sting::compareToIgnoreCase 等同于 (x,  y) -> x.compareToIgnoreCase(y);

```java
Comparator<String> comparator = String::compareTo;
```

当然还可以捕获 this 指针，this :: equals 相当于 x -> this.equals(x);


【参考资料】

1. [写给大忙人看的Java SE 8](http://book.douban.com/subject/26274206/)

---EOF---

