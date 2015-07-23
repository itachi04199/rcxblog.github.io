---
layout: post
title: java 引用介绍
categories: cache
tags: java, cache
---

## Reference

Reference 是一个抽象类，而 SoftReference，WeakReference，PhantomReference 以及 FinalReference 都是继承它的具体类。

JVM 中对象是被分配在堆（heap）上的，当程序行动中不再有引用指向这个对象时，这个对象就可以被垃圾回收器所回收。

### StrongReference

看下例子：

```java
 String tag = new String("T");
```

tag 就是一个 StrongReference。StrongReference 有一下特征：

- StrongReference 可以直接访问目标对象。
- StrongReference 所指向的对象不会被 GC。
- StrongReference 会导致内存泄露。

### SoftReference

如果一个对象只具有软引用，则内存空间足够，垃圾回收器就不会回收它；如果内存空间不足了，就会回收这些对象的内存。只要垃圾回收器没有回收它，该对象就可以被程序使用。软引用可用来实现内存敏感的高速缓存。

下面是一个代码例子：

```java
public class Bean {
	String str;
	public Bean(String string) {
		this.str = string;
	}

	@Override
	protected void finalize() throws Throwable {
		super.finalize();
		System.out.println("被回收了 bean");
	}
}

public class ReferenceTest {

	public static void main(String[] args) {
		SoftReference<Bean> reference = new SoftReference<Bean>(new Bean("rcx"));
		consumeMemory(10);
		if (reference.get() == null) {
			System.out.println("是 null 被清空");
		} else {
			System.out.println(reference.get().str);
		}
	}
	/*
    *	消耗多少 M 内存
    */
	public static void consumeMemory(int megabit) {
		String[] strings = new String[megabit];
		try {
			for (int i = 0; i < megabit; i++) {
				strings[i] = String.valueOf(new char[1024 * 1024]);
			}
		} catch (OutOfMemoryError e) {
			strings = null;
		}
	}
}
```

使用 maven 执行上面例子，并且传递 jvm 参数。

```bash
mvn exec:exec -Dexec.executable="java" -Dexec.args="-Xmx10m -Xms10m -classpath %classpath com.mycompany.app.ReferenceTest"

[INFO] --- exec-maven-plugin:1.4.0:exec (default-cli) @ my-app ---
是 null 被清空
被回收了 bean
[INFO] ------------------------------------------------------------------------

#############################

mvn exec:exec -Dexec.executable="java" -Dexec.args="-Xmx100m -Xms100m -Xmn35m -classpath %classpath com.mycompany.app.ReferenceTest"

[INFO] --- exec-maven-plugin:1.4.0:exec (default-cli) @ my-app ---
rcx
[INFO] ------------------------------------------------------------------------
```

### WeakReference

只具有弱引用的对象拥有更短暂的生命周期。在垃圾回收器线程扫描它所管辖的内存区域的过程中，一旦发现了只具有弱引用的对象，不管当前内存空间足够与否，都会回收它的内存。

弱引用可以和一个引用队列（ReferenceQueue）联合使用，如果弱引用所引用的对象被垃圾回收，Java虚拟机就会把这个弱引用加入到与之关联的引用队列中。

```java
public class ReferenceTest {

	public static void main(String[] args) {
		WeakReference<Bean> reference = new WeakReference<Bean>(new Bean("rcx"));
		consumeMemory(10);
		if (reference.get() == null) {
			System.out.println("是 null 被清空");
		} else {
			System.out.println(reference.get().str);
		}
	}

	public static void consumeMemory(int megabit) {
		String[] strings = new String[megabit];
		try {
			for (int i = 0; i < megabit; i++) {
				strings[i] = String.valueOf(new char[1024 * 1024]);
			}
		} catch (OutOfMemoryError e) {
			strings = null;
		}
	}

}
```

跟上面的例子差不多，但是执行参数 jvm 内存我们设置成 100M ，

```bash
mvn exec:exec -Dexec.executable="java" -Dexec.args="-Xmx10m -Xms10m -classpath %classpath com.mycompany.app.ReferenceTest"

[INFO] --- exec-maven-plugin:1.4.0:exec (default-cli) @ my-app ---
被回收了 bean
是 null 被清空
[INFO] ------------------------------------------------------------------------
```

### PhantomReference

"虚引用"顾名思义，就是形同虚设，与其他几种引用都不同，虚引用并不会决定对象的生命周期。如果一个对象仅持有虚引用，那么它就和没有任何引用一样，在任何时候都可能被垃圾回收器回收。

虚引用主要用来跟踪对象被垃圾回收器回收的活动。虚引用与软引用和弱引用的一个区别在于：虚引用必须和引用队列 （ReferenceQueue）联合使用。当垃圾回收器准备回收一个对象时，如果发现它还有虚引用，就会在回收对象的内存之前，把这个虚引用加入到与之关联的引用队列中。

在一个 PhantomReference 对象调用 get 方法一直返回 null。

## 总结

- 软引用 (SoftReference), 引用类型表现为当内存接近满负荷 , 或对象由 SoftReference.get() 方法的调用没有发生一段时间后 , 垃圾回收器将会清理该对象 . 在运行对象的 finalize 方法前 , 会将软引用对象加入 ReferenceQueue 中去 .
- 弱引用 (WeakReference), 引用类型表现为当系统垃圾回收器开始回收时 , 则立即会回收该对象的引用 . 与软引用一样 , 弱引用也会在运行对象的 finalize 方法之前将弱引用对象加入 ReferenceQueue.
- 强引用 (FinalReference), 这是最常用的引用类型 . JVM 系统采用 Finalizer 来管理每个强引用对象 , 并将其被标记要清理时加入 ReferenceQueue, 并逐一调用该对象的 finalize() 方法 .
- 虚引用 (PhantomReference), 这是一个最虚幻的引用类型 . 无论是从哪里都无法再次返回被虚引用所引用的对象 . 虚引用在系统垃圾回收器开始回收对象时 , 将直接调用 finalize() 方法 , 但不会立即将其加入回收队列 . 只有在真正对象被 GC 清除时 , 才会将其加入 Reference 队列中去 .


【参考资料】

1. http://blog.csdn.net/caijunjun1006/article/details/11935967
2. http://www.ibm.com/developerworks/cn/java/j-lo-langref/index.html?ca=drs-
3. http://vibexie.com/index.php/2015/05/16/understanding-of-java-strongreferencesoftreferenceweakreferencephantomreference-4-references

---EOF---
