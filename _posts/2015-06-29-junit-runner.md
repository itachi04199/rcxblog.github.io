---
layout: post
title: junit 源码学习之 Runner
categories: junit
tags: java, junit
---

## Runner 作用

Runner 是运行真正的单元测试的，并且会运行时候通知 RunNotifier 对象。如果要自己实现一个 Runner 的子类，需要提供一个构造方法并且包含一个 Class 参数。

## Runner 方法

```java
public abstract Description getDescription();

public abstract void run(RunNotifier notifier);
```

## Runner 的继承体系

![Runner 的继承体系](http://renchx.com/public/images/junit4-runner1.png)

## CompositeRunner

在 ClassesRequest 当中通过 getRunner 方法获取 Runner 的时候，我们返回的就是 CompositeRunner 对象。

从名字就可以看出来 CompositeRunner 是组合了基本的 Runner，看下内部的实现。

```java
public class CompositeRunner extends Runner implements Filterable, Sortable {
	private final List<Runner> fRunners= new ArrayList<Runner>();
	private final String fName;

	public CompositeRunner(String name) {
		fName= name;
	}

	@Override
	public void run(RunNotifier notifier) {
		for (Runner each : fRunners)
			each.run(notifier);
	}

	@Override
	public Description getDescription() {
		Description spec= Description.createSuiteDescription(fName);
		for (Runner runner : fRunners) {
			spec.addChild(runner.getDescription());
		}
		return spec;
	}

	public List<Runner> getRunners() {
		return fRunners;
	}

	public void addAll(List<? extends Runner> runners) {
		fRunners.addAll(runners);
	}

	public void add(Runner runner) {
		fRunners.add(runner);
	}

	public void filter(Filter filter) throws NoTestsRemainException {
		for (Iterator iter= fRunners.iterator(); iter.hasNext();) {
			Runner runner= (Runner) iter.next();
			if (filter.shouldRun(runner.getDescription())) {
				filter.apply(runner);
			} else {
				iter.remove();
			}
		}
	}

	protected String getName() {
		return fName;
	}

	public void sort(final Sorter sorter) {
		Collections.sort(fRunners, new Comparator<Runner>() {
			public int compare(Runner o1, Runner o2) {
				return sorter.compare(o1.getDescription(), o2.getDescription());
			}
		});
		for (Runner each : fRunners) {
			sorter.apply(each);
		}
	}
}
```

内部实现就是通过一个 List ，并且实现了 Filterable, Sortable 接口，可以进行排序和过滤，当然 run 方法的实现更加简单就是遍历 List 调用真正的 Runner 的 run 方法。

## TestClassRunner

在 ClassRequest 当中获取 runner 的时候有一个方法进行 getRunnerClass 判断当前是符合什么类型的 Runner。如果 Junit 版本 4.0 以上并且不带 @RunWith 返回的就是 TestClassRunner 类型的 Runner。

```java
public class TestClassRunner extends Runner implements Filterable, Sortable {
	protected final Runner fEnclosedRunner;
	private final Class<?> fTestClass;

	public TestClassRunner(Class<?> klass) throws InitializationError {
		this(klass, new TestClassMethodsRunner(klass));
        // 默认在 ClassRequest 当中调用的是这个构造方法，会创建一个 
        // TestClassMethodsRunner 对象
	}

	public TestClassRunner(Class<?> klass, Runner runner) throws InitializationError {
		fTestClass= klass;
		fEnclosedRunner= runner;
		MethodValidator methodValidator= new MethodValidator(klass);
		validate(methodValidator);
		methodValidator.assertValid();
	}

	// 主要检验测试类是否符合 Junit 的规范
    // 1. 包含无参数构造方法
    // 2. 带 BeforeClass 和 AfterClass 注解的方法是不是静态的
    // 3. 带 Before 和 After 和 Test 注解的方法是不是实例的
	protected void validate(MethodValidator methodValidator) {
		methodValidator.validateAllMethods();
	}

	@Override
	public void run(final RunNotifier notifier) {
		BeforeAndAfterRunner runner = new BeforeAndAfterRunner(getTestClass(),
				BeforeClass.class, AfterClass.class, null) {
			@Override
			protected void runUnprotected() {
				fEnclosedRunner.run(notifier);
			}

			// TODO: looks very similar to other method of BeforeAfter, now
			@Override
			protected void addFailure(Throwable targetException) {
				notifier.fireTestFailure(new Failure(getDescription(), targetException));
			}
		};

		runner.runProtected();
	}

	@Override
	public Description getDescription() {
		return fEnclosedRunner.getDescription();
	}

	public void filter(Filter filter) throws NoTestsRemainException {
		filter.apply(fEnclosedRunner);
	}

	public void sort(Sorter sorter) {
		sorter.apply(fEnclosedRunner);
	}

	protected Class<?> getTestClass() {
		return fTestClass;
	}
}
```

在 TestClassRunner 当中的 run 方法是，首先创建一个 BeforeAndAfterRunner 的匿名子类，然后执行 runUnprotected 方法，但是子类复写了父类的实现。至是执行 fEnclosedRunner 的 run 方法，fEnclosedRunner 是 TestClassMethodsRunner 对象实例。

## TestClassMethodsRunner

```java
public class TestClassMethodsRunner extends Runner implements Filterable, Sortable {
	private final List<Method> fTestMethods;
	private final Class<?> fTestClass;


	public TestClassMethodsRunner(Class<?> klass) {
		fTestClass= klass;
		fTestMethods= new TestIntrospector(getTestClass()).getTestMethods(Test.class);
        //获取要执行的类当中包含 @Test 注解的方法
	}

	@Override
	public void run(RunNotifier notifier) {
		if (fTestMethods.isEmpty())
			testAborted(notifier, getDescription());
		for (Method method : fTestMethods)
			invokeTestMethod(method, notifier);//执行单元测试方法
	}

	private void testAborted(RunNotifier notifier, Description description) {
		notifier.fireTestStarted(description);
		notifier.fireTestFailure(new Failure(description, new Exception("No runnable methods")));
		notifier.fireTestFinished(description);
	}

	@Override
	public Description getDescription() {
		Description spec= Description.createSuiteDescription(getName());
		List<Method> testMethods= fTestMethods;
		for (Method method : testMethods)
				spec.addChild(methodDescription(method));
		return spec;
	}

	protected String getName() {
		return getTestClass().getName();
	}

	protected Object createTest() throws Exception {
		return getTestClass().getConstructor().newInstance();
	}

	protected void invokeTestMethod(Method method, RunNotifier notifier) {
		Object test;
		try {
			test= createTest();// 创建被执行类实例，所以每次单元测试方法执行都是不同对象。
		} catch (Exception e) {
			testAborted(notifier, methodDescription(method));
			return;
		}
		createMethodRunner(test, method, notifier).run();
	}

	protected TestMethodRunner createMethodRunner(Object test, Method method, RunNotifier notifier) {
		return new TestMethodRunner(test, method, notifier, methodDescription(method));
        //创建 TestMethodRunner 对象执行
	}

	protected String testName(Method method) {
		return method.getName();
	}

	protected Description methodDescription(Method method) {
		return Description.createTestDescription(getTestClass(), testName(method));
	}

	public void filter(Filter filter) throws NoTestsRemainException {
		for (Iterator iter= fTestMethods.iterator(); iter.hasNext();) {
			Method method= (Method) iter.next();
			if (!filter.shouldRun(methodDescription(method)))
				iter.remove();
		}
		if (fTestMethods.isEmpty())
			throw new NoTestsRemainException();
	}

	public void sort(final Sorter sorter) {
		Collections.sort(fTestMethods, new Comparator<Method>() {
			public int compare(Method o1, Method o2) {
				return sorter.compare(methodDescription(o1), methodDescription(o2));
			}
		});
	}

	protected Class<?> getTestClass() {
		return fTestClass;
	}
}
```

## TestMethodRunner

```java
// 继承了 BeforeAndAfterRunner
public class TestMethodRunner extends BeforeAndAfterRunner {
	private final Object fTest;
	private final Method fMethod;
	private final RunNotifier fNotifier;
	private final TestIntrospector fTestIntrospector;
	private final Description fDescription;

	public TestMethodRunner(Object test, Method method, RunNotifier notifier, Description description) {
		super(test.getClass(), Before.class, After.class, test);
		fTest= test;
		fMethod= method;
		fNotifier= notifier;
		fTestIntrospector= new TestIntrospector(test.getClass());
		fDescription= description;
	}

	public void run() {
		if (fTestIntrospector.isIgnored(fMethod)) {
			fNotifier.fireTestIgnored(fDescription);
			return;
		}
		fNotifier.fireTestStarted(fDescription);
		try {
			long timeout= fTestIntrospector.getTimeout(fMethod);
			if (timeout > 0)
				runWithTimeout(timeout);//执行带超时时间的
			else
				runMethod();
		} finally {
			fNotifier.fireTestFinished(fDescription);
		}
	}

	private void runWithTimeout(long timeout) {
		ExecutorService service= Executors.newSingleThreadExecutor();
		Callable<Object> callable= new Callable<Object>() {
			public Object call() throws Exception {
				runMethod();
				return null;
			}
		};
		Future<Object> result= service.submit(callable);
		service.shutdown();
		try {
			boolean terminated= service.awaitTermination(timeout,
					TimeUnit.MILLISECONDS);
			if (!terminated)
				service.shutdownNow();
			result.get(timeout, TimeUnit.MILLISECONDS); // throws the exception if one occurred during the invocation
		} catch (TimeoutException e) {
			addFailure(new Exception(String.format("test timed out after %d milliseconds", timeout)));
		} catch (Exception e) {
			addFailure(e);
		}
	}

	private void runMethod() {
		runProtected();//父类当中的方法
	}

	@Override
	protected void runUnprotected() {
		try {
			executeMethodBody();
			if (expectsException())
				addFailure(new AssertionError("Expected exception: " + expectedException().getName()));
		} catch (InvocationTargetException e) {
			Throwable actual= e.getTargetException();
			if (!expectsException())
				addFailure(actual);
			else if (isUnexpected(actual)) {
				String message= "Unexpected exception, expected<" + expectedException().getName() + "> but was<"
					+ actual.getClass().getName() + ">";
				addFailure(new Exception(message, actual));
			}
		} catch (Throwable e) {
			addFailure(e);
		}
	}

	protected void executeMethodBody() throws IllegalAccessException, InvocationTargetException {
		fMethod.invoke(fTest);
	}

	@Override
	protected void addFailure(Throwable e) {
		fNotifier.fireTestFailure(new Failure(fDescription, e));
	}

	private boolean expectsException() {
		return expectedException() != null;
	}

	private Class<? extends Throwable> expectedException() {
		return fTestIntrospector.expectedException(fMethod);
	}

	private boolean isUnexpected(Throwable exception) {
		return ! expectedException().isAssignableFrom(exception.getClass());
	}
}
```

【参考源码版本】

1. Junit4.0

---EOF---
