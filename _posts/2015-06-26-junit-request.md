---
layout: post
title: junit 源码学习之 Request
categories: junit
tags: java, junit
---

## Request 作用

Request 是对将要运行的测试的抽象描述。如果我们要支持过滤和排序，我们需要一个更抽象的说明，比测试本身要更加丰富。

当运行单元测试的时候，每一个 Runner 都被 Requst 创建。Runner 会返回一个详情 Description 一个树形结构来描述将要运行的测试。

## Request 方法

Request 提供了很多静态方法，并且 Request 是一个抽象类，提供了一个抽象方法来获取 Runner。

```java
public abstract Runner getRunner();
```

一个典型的静态方法就是在 JUnitCore 当中调用的：

```java
public Result run(Class... classes) {
		return run(Request.classes("All", classes));//classes 是要被执行测试的 class 对象
}

// Request 的 classes 方法
public static Request classes(String collectionName, Class... classes) {
		return new ClassesRequest(collectionName, classes);
}
```

## Request 的继承体系

![Request 继承图](http://renchx.com/public/images/junit4-2.png)

## ClassesRequest

在 JUnitCore 当中调用 run 方法的时候是创建的 ClassesRequest，下面看下源代码：

```java
public class ClassesRequest extends Request {
	private final Class[] fClasses;
	private final String fName;

	public ClassesRequest(String name, Class... classes) {
		fClasses= classes;
		fName= name;
	}

	@Override
	public Runner getRunner() {
		CompositeRunner runner= new CompositeRunner(fName);
		for (Class<?> each : fClasses) {
			Runner childRunner= Request.aClass(each).getRunner();
			if (childRunner != null)
				runner.add(childRunner);
		}
		return runner;
	}
}
```

代码比较简单，主要有两个成员变量，fClasses (将要执行测试的 class)和 fName (描述)。

在看 getRunner 方法，会返回一个 CompositeRunner 对象。里面有段代码是如下：

```java
Runner childRunner= Request.aClass(each).getRunner();
```

aClass 方法是 Request 里面的静态方法，主要是创建了一个 ClassRequest：

```java
public static Request aClass(Class<?> clazz) {
		return new ClassRequest(clazz);
	}
```

从代码上就可以看到 ClassesRequest 的 getRunner 就是组合了每一个具体执行的 ClassRequest 当中的 Runner 对象。

## ClassRequest

Request.aClass 会返回一个 ClassRequest 对象，下面看下源码：

```java
public class ClassRequest extends Request {
	private final Class<?> fTestClass;

	public ClassRequest(Class<?> each) {
		fTestClass= each;
	}

	@Override
	public Runner getRunner() {
		Class runnerClass= getRunnerClass(fTestClass);
		try {
			Constructor constructor= runnerClass.getConstructor(Class.class); // TODO good error message if no such constructor
			Runner runner= (Runner) constructor
					.newInstance(new Object[] { fTestClass });
			return runner;
		} catch (Exception e) {
			return Request.errorReport(fTestClass, e).getRunner();
		}
	}

	Class getRunnerClass(Class<?> testClass) {
		RunWith annotation= testClass.getAnnotation(RunWith.class);
		if (annotation != null) {
			return annotation.value();
		} else if (isPre4Test(testClass)) {
			return OldTestClassRunner.class;
		} else {
			return TestClassRunner.class;
		}
	}

	boolean isPre4Test(Class<?> testClass) {
		return junit.framework.TestCase.class.isAssignableFrom(testClass);
	}
}
```

还是看 getRunner 方法，在方法当中先调用 getRunnerClass 方法来获取 runnerClass。

getRunnerClass 方法是判断将要执行的 class 的 runner 是什么类型的。如果我们的测试类上有 @RunWith 注解，就用 @RunWith 里面是值作为 runner 来运行这个测试类。另外两个判断是是否是老版本继承 TestCase 的。

然后通过反射创建 runner 对象。

## 其他 Request

ErrorReportingRequest 当出现错误的时候会返回这个 Request。并且在 Request 当中有响应的静态方法：

```java
public static Request errorReport(Class<?> klass, Throwable cause) {
		return new ErrorReportingRequest(klass, cause);
	}
```

FilterRequest 和 SortingRequest 都是包装类型对其他类型的 Request 进行了包装，从而提供了过滤和排序的功能。

【参考源码版本】

1. Junit4.0

---EOF---
