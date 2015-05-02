---
layout: post
title: ComparisonChain 源码学习
categories: guava
tags: 笔记 java
---

ComparisonChain 类用来简化 compare 操作的的工具类。

ComparisonChain 是一个抽象类，提供了链式的比较方式。

首先看下一般的使用方式：

```java
ComparisonChain.start().compare(age, o.age).compare(name, o.name).result();
```

再来看下内部 start 方法的定义：

```java
public static ComparisonChain start() {
    return ACTIVE;
  }

  private static final ComparisonChain ACTIVE = new ComparisonChain() {
    @SuppressWarnings("unchecked")
    @Override public ComparisonChain compare(
        Comparable left, Comparable right) {
      return classify(left.compareTo(right));
    }
    @Override public <T> ComparisonChain compare(
        @Nullable T left, @Nullable T right, Comparator<T> comparator) {
      return classify(comparator.compare(left, right));
    }
    @Override public ComparisonChain compare(int left, int right) {
      return classify(Ints.compare(left, right));
    }
    @Override public ComparisonChain compare(long left, long right) {
      return classify(Longs.compare(left, right));
    }
    @Override public ComparisonChain compare(float left, float right) {
      return classify(Float.compare(left, right));
    }
    @Override public ComparisonChain compare(double left, double right) {
      return classify(Double.compare(left, right));
    }
    @Override public ComparisonChain compareTrueFirst(boolean left, boolean right) {
      return classify(Booleans.compare(right, left)); // reversed
    }
    @Override public ComparisonChain compareFalseFirst(boolean left, boolean right) {
      return classify(Booleans.compare(left, right));
    }
    ComparisonChain classify(int result) {
      return (result < 0) ? LESS : (result > 0) ? GREATER : ACTIVE;
    }
    @Override public int result() {
      return 0;
    }
  };
```

自己内部有一个 ACTIVE 的匿名内部类变量，里面实现了 ComparisonChain 当中的 compare 方法，基本上都是使用 Guava 当中的工具类来真正执行 compare 方法，每个 compare 方法都调用了 classify 方法。

classify 方法很关键，里面的逻辑是判断传入的参数是否大于 0，小于 0，或者等于 0。

如果小于 0，则代表右边大，会返回 LESS 对象，因为是一个链式的调用过程，当分出大小后就不需要执行下面的比较方法，所以 LESS 对象的定义是如下的：

```java
private static final ComparisonChain LESS = new InactiveComparisonChain(-1);

private static final class InactiveComparisonChain extends ComparisonChain {
    final int result;

    InactiveComparisonChain(int result) {
      this.result = result;
    }
    @Override public ComparisonChain compare(
        @Nullable Comparable left, @Nullable Comparable right) {
      return this;
    }
    @Override public <T> ComparisonChain compare(@Nullable T left,
        @Nullable T right, @Nullable Comparator<T> comparator) {
      return this;
    }
    @Override public ComparisonChain compare(int left, int right) {
      return this;
    }
    @Override public ComparisonChain compare(long left, long right) {
      return this;
    }
    @Override public ComparisonChain compare(float left, float right) {
      return this;
    }
    @Override public ComparisonChain compare(double left, double right) {
      return this;
    }
    @Override public ComparisonChain compareTrueFirst(boolean left, boolean right) {
      return this;
    }
    @Override public ComparisonChain compareFalseFirst(boolean left, boolean right) {
      return this;
    }
    @Override public int result() {
      return result;
    }
  }
```

InactiveComparisonChain 是继承了 ComparisonChain 的子类。里面有一个成员变量 result 就是整个链调用最后返回的结果，再来看他们 compare 方法都会返回 this 本身，所以就不会调用分出大小的后面的 compare 方法。

所以大于 0 的情况也相似，只是传入的参数不同：

```java
private static final ComparisonChain GREATER = new InactiveComparisonChain(1);
```

中有当等于 0 的时候才回返回 ACTIVE，真正的比较下一个 compare 方法。

当之前没看到源码的时候，我也很难解为什么链式调用还能中断返回值，原来是通过这样的实现，让看似链式调用的形式，其实已经在分出大小的时候已经中断，并且有了返回结果。这样链式的实现方式，很好的避免了 if else 的判断，使代码清晰了很多。

---EOF---

