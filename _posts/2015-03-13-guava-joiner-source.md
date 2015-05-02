---
layout: post
title: Joiner 源码学习
categories: guava
tags: 笔记 java
---

首先看如何创建 Joiner 对象，Joiner 的构造方法是私有的，只能通过静态方法来创建 Joiner 对象，并且 Joiner 是一个不可变类。

```java
public static Joiner on(String separator) {
    return new Joiner(separator);
  }

  public static Joiner on(char separator) {
    return new Joiner(String.valueOf(separator));
  }

  private final String separator;

  private Joiner(String separator) {
    this.separator = checkNotNull(separator);
  }

  private Joiner(Joiner prototype) {
    this.separator = prototype.separator;
  }
```

Joiner 可以通过 on 方法传入一个分隔符参数来创建 Joiner 对象。

一般 Joiner 的使用方式如下：

```java
String string = Joiner.on("|").skipNulls().join(list);
String expected = "rcx|rcx1|rcx2";

String string1 = Joiner.on("|").useForNull("null").join(list);
String expected1 = "rcx|rcx1|null|rcx2";

assertEquals(expected, string);
assertEquals(expected1, string1);
```

首先看四个重载的 join 方法：

```java
  public final String join(Iterable<?> parts) {
    return join(parts.iterator());
  }

  public final String join(Iterator<?> parts) {
    return appendTo(new StringBuilder(), parts).toString();
  }

  public final String join(Object[] parts) {
    return join(Arrays.asList(parts));
  }

  public final String join(@Nullable Object first, @Nullable Object second, Object... rest) {
    return join(iterable(first, second, rest));
  }
```

可以看到最后都是调用到第二个重载方法上面，前三个比较简单，我们分析下最后一个，需要分析下 iterable 方法：

```java
private static Iterable<Object> iterable(
      final Object first, final Object second, final Object[] rest) {
    checkNotNull(rest);
    return new AbstractList<Object>() {
      @Override public int size() {
        return rest.length + 2;
      }

      @Override public Object get(int index) {
        switch (index) {
          case 0:
            return first;
          case 1:
            return second;
          default:
            return rest[index - 2];
        }
      }
    };
  }
```

这里的 iterable 方法，它把两个变量和一个数组变成了一个实现了 Iterable 接口的集合，手法精妙！

看下 AbstractList 中迭代器代码的实现：

```java
public boolean hasNext() {
    return cursor != size();
}
public E next() {
    checkForComodification();
    try {
        int i = cursor;
        E next = get(i);
        lastRet = i;
        cursor = i + 1;
        return next;
    } catch (IndexOutOfBoundsException e) {
        checkForComodification();
        throw new NoSuchElementException();
    }
}
```

hasNext 中关键的函数调用是 size，获取集合的大小。next 方法中关键的函数调用是 get，获取第 i 个元素。Guava 的实现返回了一个被覆盖了 size 和 get 方法的 AbstractList，巧妙的复用了由编译器生成的数组，避免了新建列表和增加元素的开销。

下面就分析调用的 appendTo 方法，这个 appendTo 方法有八个重载的，如下：

```java
public <A extends Appendable> A appendTo(A appendable, Iterable<?> parts) throws IOException {
    return appendTo(appendable, parts.iterator());
  }

  public <A extends Appendable> A appendTo(A appendable, Iterator<?> parts) throws IOException {
    checkNotNull(appendable);
    if (parts.hasNext()) {
      appendable.append(toString(parts.next()));
      while (parts.hasNext()) {
        appendable.append(separator);
        appendable.append(toString(parts.next()));
      }
    }
    return appendable;
  }

  public final <A extends Appendable> A appendTo(A appendable, Object[] parts) throws IOException {
    return appendTo(appendable, Arrays.asList(parts));
  }

  public final <A extends Appendable> A appendTo(
      A appendable, @Nullable Object first, @Nullable Object second, Object... rest)
          throws IOException {
    return appendTo(appendable, iterable(first, second, rest));
  }

  public final StringBuilder appendTo(StringBuilder builder, Iterable<?> parts) {
    return appendTo(builder, parts.iterator());
  }

  public final StringBuilder appendTo(StringBuilder builder, Iterator<?> parts) {
    try {
      appendTo((Appendable) builder, parts);
    } catch (IOException impossible) {
      throw new AssertionError(impossible);
    }
    return builder;
  }

  public final StringBuilder appendTo(StringBuilder builder, Object[] parts) {
    return appendTo(builder, Arrays.asList(parts));
  }

  public final StringBuilder appendTo(
      StringBuilder builder, @Nullable Object first, @Nullable Object second, Object... rest) {
    return appendTo(builder, iterable(first, second, rest));
  }
```

可以看到最后都调用到了下面这个 appendTo 方法上面：

```java
public <A extends Appendable> A appendTo(A appendable, Iterator<?> parts) throws IOException {
    checkNotNull(appendable);
    if (parts.hasNext()) {
      appendable.append(toString(parts.next()));
      while (parts.hasNext()) {
        appendable.append(separator);
        appendable.append(toString(parts.next()));
      }
    }
    return appendable;
  }
```

这段代码先用 if 然后嵌套个 while 循环，巧妙的避免了最后在末尾插入分隔符的尴尬；第二个技巧是使用了自定义的 toString 方法而不是 Object#toString 来将对象序列化成字符串，为后续的各种空指针保护开了方便之门。

Joiner 对 null 有两种处理方式，一种是跳过 null 元素，另一种是使用传入的字符串来代替 null。

下面看使用字符串来代替 null

```java
  public Joiner useForNull(final String nullText) {
    checkNotNull(nullText);
    return new Joiner(this) {
      @Override CharSequence toString(@Nullable Object part) {
        return (part == null) ? nullText : Joiner.this.toString(part);
      }

      @Override public Joiner useForNull(String nullText) {
        checkNotNull(nullText); // weird: just to satisfy NullPointerTester.
        throw new UnsupportedOperationException("already specified useForNull");
      }

      @Override public Joiner skipNulls() {
        throw new UnsupportedOperationException("already specified useForNull");
      }
    };
  }
```

从这个方法就可以看出来 Joiner 是个不可变类，因为这个 useForNull 方法返回了一个新的 Joiner 对象。

首先是使用复制构造函数保留先前初始化时候设置的分隔符，然后覆盖了之前提到的 toString 方法。为了防止重复调用 useForNull 和 skipNulls，还特意覆盖了这两个方法，一旦调用就抛出运行时异常。为什么不能重复调用 useForNull ？因为覆盖了 toString 方法，而覆盖实现中需要调用覆盖前的 toString。

在看下 skipNulls  方法的实现：

```java
public Joiner skipNulls() {
    return new Joiner(this) {
      @Override public <A extends Appendable> A appendTo(A appendable, Iterator<?> parts)
          throws IOException {
        checkNotNull(appendable, "appendable");
        checkNotNull(parts, "parts");
        while (parts.hasNext()) {
          Object part = parts.next();
          if (part != null) {
            appendable.append(Joiner.this.toString(part));
            break;
          }
        }
        while (parts.hasNext()) {
          Object part = parts.next();
          if (part != null) {
            appendable.append(separator);
            appendable.append(Joiner.this.toString(part));
          }
        }
        return appendable;
      }

      @Override public Joiner useForNull(String nullText) {
        checkNotNull(nullText); // weird: just to satisfy NullPointerTester.
        throw new UnsupportedOperationException("already specified skipNulls");
      }

      @Override public MapJoiner withKeyValueSeparator(String kvs) {
        checkNotNull(kvs); // weird: just to satisfy NullPointerTester.
        throw new UnsupportedOperationException("can't use .skipNulls() with maps");
      }
    };
  }
```

skipNulls 的实现就相对要复杂一些，覆盖了原先全功能 appendTo 中使用 if 和 while 的优雅实现，变成了 2 个 while 先后执行。第一个 while 找到 第一个不为空指针的元素，起到之前的 if 的功能，第二个 while 功能和之前的一致。

MapJoiner 实现为 Joiner 的一个静态内部类，它的构造函数和 Joiner 一样也是私有，只能通过 Joiner#withKeyValueSeparator 来生成实例。类似地，MapJoiner 也实现了 appendTo 方法和一系列的重载，还用 join 方法对 appendTo 做了封装。MapJoiner 整个实现和 Joiner 大同小异，在实现中大量使用 Joiner 的 toString 方法来保证空指针保护行为和初始化时的语义一致。

MapJoiner 也实现了一个 useForNull 方法，这样的好处是，在获取 MapJoiner 之后再去设置空指针保护，和获取 MapJoiner 之前就设置空指针保护，是等价的，用户无需去关心顺序问题。


【参考资料】

1. [Guava 是个风火轮之基础工具(1)](http://www.importnew.com/15221.html)

---EOF---

