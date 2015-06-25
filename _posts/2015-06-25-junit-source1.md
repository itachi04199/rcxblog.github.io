---
layout: post
title: junit 框架深入学习
categories: junit
tags: java, junit
---

## 入口

在命令行执行 Junit

```bash
java -cp .:junit-4.XX.jar:hamcrest-core-1.3.jar org.junit.runner.JUnitCore AppTest
```

在 IDE 当中执行 Junit，如 eclipse 集成了 Junit 插件

![执行时序图](http://renchx.com/public/images/junit4-1.png)

## JUnitCore

JUnitCore 用在命令行当中执行 Junit 单元测试，可以使用两种形式：一种是执行 main 方法，另外一种是执行 JUnitCore 的静态方法 runClasses。

## Test runners

一般 Junit4 以上版本写一个简单的单元测试如下：

```java
public class AppTest {

	private App app;

	@Before
	public void setUp() {
		app = new App();
	}

	@Test
	public void testAdd() {
		int actual = app.add(3, 4);
		int expected = 7;
		Assert.assertEquals("this is error testAdd", expected, actual);
	}

}
```

如果我们执行这个单元测试，就会把这个类的带 @Test 注解的方法封装成 Runner，然后来执行。

## Request

Client 会根据测试类或方法而创建一个 Request 实体，Request 包含了要被执行的测试类、测试方法等信息。

## RunNotifier

在执行 Runner.run 方法时 Client 还会传递一个 RunNotifier 对象，是事件的监听器的管理员。Runner 在开始执行、成功、失败和执行结束时会调用 RunNotifier 中相应的方法从而发送事件给注册了的监听器，JUnit运行的最终结果就是这些监听器收集展现的。

## Result

单元测试执行完的结果，可以通过这个对象来描述当前单元测试的执行情况。

---EOF---

