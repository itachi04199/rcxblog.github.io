---
layout: post
title: Java8 lambda 表达式初体验下
categories: java基础
tags: 笔记, java
---

## 构造器引用

构造器引用与方法引类似，不同的是构造器引用使用的方法名是 new。例如，Buttton::new。

```java
List<String> strings = new ArrayList<String>();
strings.add("a");
strings.add("b");
strings.add("c");
Stream<Button> stream = strings.stream().map(Button::new);
List<Button> buttons = stream.collect(Collectors.toList());
```

先不详细介绍 stream map collect 方法，主要看对于每个列表元素会调用 Button 的构造方法。虽然 Button 有多个构造器，但是会选择只有一个 String 参数的构造器。

数组类型的构造器引用，int[]::new 是一个含有一个参数的构造器引用，这个参数就是数组的长度，相当于 x -> new int[x]。

## 变量作用域

有如下代码：

```java
public static void repeat(String string, int count) {
		Runnable runnable = () -> {
			for (int i = 0; i < count; i++) {
				System.out.println(this.toString());
                Thread.yield();
			}
		};
		new Thread(runnable).start();
	}
```

上面这段代码的两个参数没有设置成 final 的，这在 JDK7 之前是会编译错误的，同样在 java8 当中匿名内部类访问外部也不需要 final 来修饰。

分析下上面的代码，由于有 Thread.yield 所以可能其他线程占用 CPU 先执行，然后方法 repeat() 先反回了，才执行 runnable，那么这个时候 string 和 count 这 2 个参数怎么办？

首先一个 lambda 表达式需要有三个部分：

- 一段代码
- 参数
- 自由变量的值，这里的“自由”指的是那些不是传入表达式的参数并且没有在代码中定义的变量。

上面的那个例子当中有两个自由变量，string 和 count，lambad 表达式必须存放这两个变量的值。并且含有自由变量的代码块被称为闭包。

在 lambda 表达式当中被引用的变量的值不可以被更改，编译器会检查修改操作：

```java
public void repeat(String string, int count) {
		Runnable runnable = () -> {
			for (int i = 0; i < count; i++) {
				string = string + "a";//编译出错
				System.out.println(this.toString());
			}
		};
		new Thread(runnable).start();
	}
```

在 lambda 表达式当中不允许声明一个与局部变量同名的参数或者局部变量。

```java
String first = "";
Comparator<String> comparator = (first, second) -> Integer.compare(first.length(),//编译会出错
				second.length());
```

lambda 表达式中使用 this 会引用创建该 lambda 表达式的方法的 this 参数，

```java
public class Testmain2 {
	public static void main(String[] args) {
		Testmain2 testmain2 = new Testmain2();
		testmain2.method();
	}

	@Override
	public String toString() {
		return "aaaa";
	}

	public void method() {
		Runnable runnable = () -> {
			System.out.println(this.toString());
		};
		new Thread(runnable).start();
	}
}
```

上面的例子执行后会输出：aaaa。

## 默认方法

在集合库当中提供了一些函数表达式，例如 forEach 方法：

```java
list.forEach(System.out::println);
```

由于集合的接口是之前定义的，新添加一个 forEach 方法会导致老的代码不兼容，但是 java8 当中是给接口设计成可以包含具体实现的默认方法来解决这个问题。

```java
public interface Person {
	long getID();

	default String getName() {
		return "name";
	}
}
```

如果要实现 Person 接口，那么必须实现 getID 方法，getName 方法可以不实现。

如果一个接口定义了一个默认方法，而另外一个父类中又定义了同名的方法，那么如何选择？有以下规则：

1. 选择父类中的方法，如果父类提供了具体的实现方式，那么接口中具有相同名称和参数的默认方法会被忽略。
2. 接口冲突，如果需要实现两个接口，并且这两个接口有两个相同签名的默认方法，那么子类就需要覆盖重写这个方法。

如果一个子类继承了一个父类，并且实现了一个接口，并且父类和接口有相同签名的默认方法，那么之类继承父类当中的实现，类优先可以保持 java7 的兼容性。

**注意**，不能为 Object 中的方法重新定义个默认方法。

## 接口中的静态方法

java8 可以在接口当中添加静态方法，便于把一些工具方法加入到接口当中，所以类似一些， Collections 和 Paths 类比较尴尬。

【参考资料】

1. [写给大忙人看的Java SE 8](http://book.douban.com/subject/26274206/)

---EOF---
