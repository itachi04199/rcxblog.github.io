---
layout: post
title: effective-java类和接口1
categories: effective-java
tags: 笔记
---

#### 使类和成员的可访问性最小化

设计良好的模块会隐藏所有的实现细节，把它的 API 与它的实现清晰的隔离出来。

**尽可能使每个类或者成员不被外界访问。**对于顶层的类和接口，只有两种可能的访问级别，包级别的和公有的。如果是公有的，那它就是 API 的一部分，如果是包级别的那么它可以在以后的版本中对它进行修改。

如果一个包级别的类或接口只在某一个类中使用，那么可以考虑使它成为哪个类的私有嵌套类。

有一条规则限制了降低方法的可访问性的能力。如果方法覆盖了超类中的一个方法，子类中的访问级别就不允许低于超类中的访问级别。这样可以确保任何可以使用超类的实例的地方也都可以使用子类的实例。

**实例域不能是公有的。**如果实例域是公有的那么就相当于放弃了对这个域的值进行限制的能力。并且，包含公有可变域的类是线程不安全的。

#### 在公有类中使用访问方法而非公有域

看下面的类：

```java
public class Point {
	public int x;
    public int y;
}
```

如果公有类暴露了它的数据域，要想在将来改变其内部表示法是不可能的，应该使用 getter 和 setter 方法。

如果类是包级别私有的，或者是私有嵌套类，直接暴露它的数据域并没有本质的错误。

#### 使可变性最小化

不可变类比可变类更加易于设计、使用和实现。将类变成不可变的，要遵循下面规则：

- 不提供任何修改对象状态的方法。
- 保证类不能被扩展。一般类设计成 final 或者私有构造器。
- 所有域都是 final 的，并且是私有的。
- 确保对于任何可变组件的互斥访问。如果类具有指向可变对象的域，则必须确保该类的客户端无法获得指向这些对象的引用。

可以参见[这里](http://renchx.com/java-concurrenct-learn2/)定义不可变对象的策略。

不可变类的缺点是，对于每个不同的值都需要一个单独的对象。在操作很多不可变对象的时候会带来性能的问题。

#### 复合优先于继承

与方法调用不同的是，继承打破了封装性。如果，子类依赖超类中特定功能的实现细节。超类实现改变的话，子类会受到影响。

看如下例子，我们想要设计一个 Set 带有查看添加了多少元素的功能。

```java
public class InstrumentedHashSet<E> extends HashSet<E> {
	private int addCount = 0;

    public InstrumentedHashSet() {
    }

    public InstrumentedHashSet(in initCap, float loadFactor) {
    	super(initCap, loadFactor);
    }

    public boolean add(E e) {
   		addCount++;
    	return super.add(e);
    }

    public boolean addAll(Collection<? extends E> c) {
   		addCount = addCount + c.size();
    	return super.addAll(C);
    }

    public int getAddCount() {
    	return addCount;
    }
}
```

当我们调用的时候：

```java
InstrumentedHashSet<String> s = new InstrumentedHashSet<String>();
s.addAll(Arrays.asList("a", "b", "c"));
```

这个 getAddCount 方法将会返回 6。这是因为 HashSet 内部 addAll 方法是基于 add 方法来实现的，代码如下。

```java
public boolean addAll(Collection<? extends E> c) {
        boolean modified = false;
        for (E e : c)
            if (add(e))
                modified = true;
        return modified;
    }
```

上面的问题可以使用复合来解决。看如下的实现：

```java
public class InstrumentedSet<E> extends ForwardingSet<E> {
	private int addCount = 0;

	public InstrumentedSet(Set<E> s) {
		super(s);
	}

	@Override
	public boolean add(E e) {
		addCount++;
		return super.add(e);
	}

	@Override
	public boolean addAll(Collection<? extends E> c) {
		addCount += c.size();
		return super.addAll(c);
	}

	public int getAddCount() {
		return addCount;
	}

	public static void main(String[] args) {
		InstrumentedSet<String> s = new InstrumentedSet<String>(
				new HashSet<String>());
		s.addAll(Arrays.asList("Snap", "Crackle", "Pop"));
		System.out.println(s.getAddCount());
	}
}

public class ForwardingSet<E> implements Set<E> {
	private final Set<E> s;

	public ForwardingSet(Set<E> s) {
		this.s = s;
	}

	public void clear() {
		s.clear();
	}

	public boolean contains(Object o) {
		return s.contains(o);
	}

	public boolean isEmpty() {
		return s.isEmpty();
	}

	public int size() {
		return s.size();
	}

	public Iterator<E> iterator() {
		return s.iterator();
	}

	public boolean add(E e) {
		return s.add(e);
	}

	public boolean remove(Object o) {
		return s.remove(o);
	}

	public boolean containsAll(Collection<?> c) {
		return s.containsAll(c);
	}

	public boolean addAll(Collection<? extends E> c) {
		return s.addAll(c);
	}

	public boolean removeAll(Collection<?> c) {
		return s.removeAll(c);
	}

	public boolean retainAll(Collection<?> c) {
		return s.retainAll(c);
	}

	public Object[] toArray() {
		return s.toArray();
	}

	public <T> T[] toArray(T[] a) {
		return s.toArray(a);
	}

	@Override
	public boolean equals(Object o) {
		return s.equals(o);
	}

	@Override
	public int hashCode() {
		return s.hashCode();
	}

	@Override
	public String toString() {
		return s.toString();
	}
}
```

InstrumentedSet 类把每一个 Set 实例都包装了起来，相当于是一个包装类。

只有当子类真正是超类的子类型时，才适合用继承。如果在应该使用复合的地方使用继承，则会不必要的暴露实现细节。

【参考资料】

1. [Effective Java](http://book.douban.com/subject/3360807/)

---EOF---

