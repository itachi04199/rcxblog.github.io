---
layout: post
title: Splitter 源码学习
categories: guava
tags: 笔记 java
---

Splitter 类有四个成员变量：

```java
  private final CharMatcher trimmer;
  private final boolean omitEmptyStrings;
  private final Strategy strategy;
  private final int limit;
```

strategy 用于帮助实现策略模式，omitEmptyStrings 用于控制是否删除拆分结果中的空字符串，通过 Splitter#omitEmptyStrings 设置，trimmer 用于描述删除拆分结果的前后空白符的策略，通过 Splitter#trimResults 设置，limit 用于控制拆分的结果个数，通过 Splitter#limit 设置。

Splitter 有两个私有的构造方法：

```java
private Splitter(Strategy strategy) {
    this(strategy, false, CharMatcher.NONE, Integer.MAX_VALUE);
  }

  private Splitter(Strategy strategy, boolean omitEmptyStrings,
      CharMatcher trimmer, int limit) {
    this.strategy = strategy;
    this.omitEmptyStrings = omitEmptyStrings;
    this.trimmer = trimmer;
    this.limit = limit;
  }
```

看最常用的 on 方法定义：

```java
public static Splitter on(final String separator) {
    checkArgument(separator.length() != 0,
        "The separator may not be the empty string.");

    return new Splitter(new Strategy() {
      @Override public SplittingIterator iterator(
          Splitter splitter, CharSequence toSplit) {
        return new SplittingIterator(splitter, toSplit) {
          @Override public int separatorStart(int start) {
            int delimeterLength = separator.length();

            positions:
            for (int p = start, last = toSplit.length() - delimeterLength;
                p <= last; p++) {
              for (int i = 0; i < delimeterLength; i++) {
                if (toSplit.charAt(i + p) != separator.charAt(i)) {
                  continue positions;
                }
              }
              return p;
            }
            return -1;
          }

          @Override public int separatorEnd(int separatorPosition) {
            return separatorPosition + separator.length();
          }
        };
      }
    });
  }
```

这个 on 方法调用第一个构造方法创建 Splitter 对象，里面实现了一个匿名 Strategy 类。实现了 iterator 方法，并且返回 SplittingIterator 类型的对象。

在看下 Strategy 接口的定义：

```java
private interface Strategy {
  Iterator<String> iterator(Splitter splitter, CharSequence toSplit);
}
```

看 split 方法的定义：

```java
public Iterable<String> split(final CharSequence sequence) {
    checkNotNull(sequence);

    return new Iterable<String>() {
      @Override public Iterator<String> iterator() {
        return spliterator(sequence);
      }
      @Override public String toString() {
        return Joiner.on(", ")
            .appendTo(new StringBuilder().append('['), this)
            .append(']')
            .toString();
      }
    };
  }

  private Iterator<String> spliterator(CharSequence sequence) {
    return strategy.iterator(this, sequence);
  }
```

split 方法返回 Iterable 对象，获取迭代器 iterator 是通过 spliterator 方法，这方法里面则是通过 Splitter 内部的 Strategy 的
iterator 方法来获取。

我们在来看 SplittingIterator 的声明：

```java
private abstract static class SplittingIterator extends AbstractIterator<String>
```

首先看 AbstractIterator 的定义：

```java
abstract class AbstractIterator<T> implements Iterator<T> {
  private State state = State.NOT_READY;

  protected AbstractIterator() {}

  private enum State {
    READY, NOT_READY, DONE, FAILED,
  }

  private T next;

  protected abstract T computeNext();

  protected final T endOfData() {
    state = State.DONE;
    return null;
  }

  @Override
  public final boolean hasNext() {
    checkState(state != State.FAILED);
    switch (state) {
      case DONE:
        return false;
      case READY:
        return true;
      default:
    }
    return tryToComputeNext();
  }

  private boolean tryToComputeNext() {
    state = State.FAILED; // temporary pessimism
    next = computeNext();
    if (state != State.DONE) {
      state = State.READY;
      return true;
    }
    return false;
  }

  @Override
  public final T next() {
    if (!hasNext()) {
      throw new NoSuchElementException();
    }
    state = State.NOT_READY;
    return next;
  }

  @Override public final void remove() {
    throw new UnsupportedOperationException();
  }
}
```

AbstractIterator 迭代器是惰性迭代器，可以看到在 AbstractIterator 内部有个私有的 State 枚举类型。

从 AbstractIterator 类中的 hasNext 方法可以看出来，当状态是 DONE 代表没元素了，状态是 READY 代表元素准备完成，NOT_READY 状态执行 tryToComputeNext 方法。

在 tryToComputeNext 方法当中，通过抽象方法 computeNext 获取到下一个元素，并且如果状态不是 DONE 把状态设置成 READY。

由于 state 是私有变量，而迭代是否结束只有在调用 computeNext 的过程中才知道，于是我们有了一个保护的 endOfData 方法，允许 AbstractIterator 的子类将 state 设置为 DONE。

我们来看 SplittingIterator 当中的 computeNext 方法：

```java
@Override protected String computeNext() {

      int nextStart = offset;
      while (offset != -1) {
        int start = nextStart;
        int end;

        int separatorPosition = separatorStart(offset);
        if (separatorPosition == -1) {
          end = toSplit.length();
          offset = -1;
        } else {
          end = separatorPosition;
          offset = separatorEnd(separatorPosition);
        }
        if (offset == nextStart) {
          offset++;
          if (offset >= toSplit.length()) {
            offset = -1;
          }
          continue;
        }

        while (start < end && trimmer.matches(toSplit.charAt(start))) {
          start++;
        }
        while (end > start && trimmer.matches(toSplit.charAt(end - 1))) {
          end--;
        }

        if (omitEmptyStrings && start == end) {
          nextStart = offset;
          continue;
        }

        if (limit == 1) {
          end = toSplit.length();
          offset = -1;
          while (end > start && trimmer.matches(toSplit.charAt(end - 1))) {
            end--;
          }
        } else {
          limit--;
        }

        return toSplit.subSequence(start, end).toString();
      }
      return endOfData();
    }
```

在 computeNext 这个方法内部维护着两个全局变量：

```java
int offset = 0;
int limit;
```

这个方法进入 while 循环后 (offsed 不为 -1 进入循环)，先通过 separatorStart 方法来找到分隔符的绝对位置。如果分隔符的绝对位置为 -1 即未找到，把 offsed 设置成 -1，如果不未 -1，把 分隔符的位置索引赋值给 end。

在 while 循环内部有两个变量，start 和 end，一个是代表截取的字符串开头索引和结尾索引。

所以这个方法的大致想法就是，通过 offset 来指定下次查找字符串当中分隔符时候从哪个索引开始，如果 offset 为 -1 说明字符串分隔完成。

下面看 separatorStart 抽象方法的实现：

```java
          @Override public int separatorStart(int start) {
            int delimeterLength = separator.length();

            positions:
            for (int p = start, last = toSplit.length() - delimeterLength;
                p <= last; p++) {
              for (int i = 0; i < delimeterLength; i++) {
                if (toSplit.charAt(i + p) != separator.charAt(i)) {
                  continue positions;
                }
              }
              return p;
            }
            return -1;
```

方法先计算分隔符的长度，然后通过参数传入进来的 start 位置开始查找，内层 for 循环是判断当前位置往后分隔符长度中每个字符是否都与分隔符一样，如果有一个不一样的就继续执行外层循环。

再来看看 separatorEnd 方法

```java
@Override public int separatorEnd(int separatorPosition) {
            return separatorPosition + separator.length();
          }
```

这个方法比较容易懂，就是返回下次开始的位置索引，即这次分隔符开始位置加上分隔符长度。

总体上来说，这就是 Splitter 主要的代码了。采用了模板模式和策略模式，对 for 循环的使用也是值得我们学习的，还有对匿名类的使用方式，以及惰性迭代器的使用。其余的方法就不仔细分析了。

【参考资料】

1. [Guava 是个风火轮之基础工具(2)](http://www.importnew.com/15227.html)

---EOF---

