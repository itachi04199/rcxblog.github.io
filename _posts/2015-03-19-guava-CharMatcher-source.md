---
layout: post
title: CharMatcher 源码学习
categories: guava
tags: 笔记 java
---

CharMatcher 是一个抽象类，里面唯一的抽象方法是 matches 方法：

```java
public abstract class CharMatcher implements Predicate<Character> {
...
public abstract boolean matches(char c);
}
```

CharMatcher 内部有许多匿名的 CharMatcher 常量对象。

```java
public static final CharMatcher BREAKING_WHITESPACE = new CharMatcher() {
    @Override
    public boolean matches(char c) {
      switch (c) {
        case '\t':
        case '\n':
        case '\013':
        case '\f':
        case '\r':
        case ' ':
        case '\u0085':
        case '\u1680':
        case '\u2028':
        case '\u2029':
        case '\u205f':
        case '\u3000':
          return true;
        case '\u2007':
          return false;
        default:
          return c >= '\u2000' && c <= '\u200a';
      }
    }
```

还有类似的如 ASCII，JAVA_DIGIT，DIGIT 等

CharMatcher 内部有三个处理与或非逻辑的私有静态内部类：

```java
private static class And extends CharMatcher {
    final CharMatcher first;
    final CharMatcher second;

    And(CharMatcher a, CharMatcher b) {
      this(a, b, "CharMatcher.and(" + a + ", " + b + ")");
    }

    And(CharMatcher a, CharMatcher b, String description) {
      super(description);
      first = checkNotNull(a);
      second = checkNotNull(b);
    }

    @Override
    public boolean matches(char c) {
      return first.matches(c) && second.matches(c);
    }

    @GwtIncompatible("java.util.BitSet")
    @Override
    void setBits(BitSet table) {
      BitSet tmp1 = new BitSet();
      first.setBits(tmp1);
      BitSet tmp2 = new BitSet();
      second.setBits(tmp2);
      tmp1.and(tmp2);
      table.or(tmp1);
    }

    @Override
    CharMatcher withToString(String description) {
      return new And(first, second, description);
    }
  }




private static class Or extends CharMatcher {
    final CharMatcher first;
    final CharMatcher second;

    Or(CharMatcher a, CharMatcher b, String description) {
      super(description);
      first = checkNotNull(a);
      second = checkNotNull(b);
    }

    Or(CharMatcher a, CharMatcher b) {
      this(a, b, "CharMatcher.or(" + a + ", " + b + ")");
    }

    @GwtIncompatible("java.util.BitSet")
    @Override
    void setBits(BitSet table) {
      first.setBits(table);
      second.setBits(table);
    }

    @Override
    public boolean matches(char c) {
      return first.matches(c) || second.matches(c);
    }

    @Override
    CharMatcher withToString(String description) {
      return new Or(first, second, description);
    }
  }





  private static class NegatedMatcher extends CharMatcher {
    final CharMatcher original;

    NegatedMatcher(String toString, CharMatcher original) {
      super(toString);
      this.original = original;
    }

    NegatedMatcher(CharMatcher original) {
      this(original + ".negate()", original);
    }

    @Override public boolean matches(char c) {
      return !original.matches(c);
    }

    @Override public boolean matchesAllOf(CharSequence sequence) {
      return original.matchesNoneOf(sequence);
    }

    @Override public boolean matchesNoneOf(CharSequence sequence) {
      return original.matchesAllOf(sequence);
    }

    @Override public int countIn(CharSequence sequence) {
      return sequence.length() - original.countIn(sequence);
    }

    @GwtIncompatible("java.util.BitSet")
    @Override
    void setBits(BitSet table) {
      BitSet tmp = new BitSet();
      original.setBits(tmp);
      tmp.flip(Character.MIN_VALUE, Character.MAX_VALUE + 1);
      table.or(tmp);
    }

    @Override public CharMatcher negate() {
      return original;
    }

    @Override
    CharMatcher withToString(String description) {
      return new NegatedMatcher(description, original);
    }
  }
```

这三个类看着很多其实想法都很简单。

还有几个静态类，RangesMatcher 和 FastMatcher、NegatedFastMatcher、BitSetMatcher.

这几个类也比较简单，RangesMatcher 是处理范围匹配的，一般可以进行预处理来提升性能，FastMatcher 类是一般来处理不能直接预处理的字符的。

常量 DIGIT 就是 RangesMatcher 的实例。

看下 CharMatcher 中的 is 方法：

```java
public static CharMatcher is(final char match) {
    String description = "CharMatcher.is('" + showCharacter(match) + "')";
    return new FastMatcher(description) {
      @Override public boolean matches(char c) {
        return c == match;
      }

      @Override public String replaceFrom(CharSequence sequence, char replacement) {
        return sequence.toString().replace(match, replacement);
      }

      @Override public CharMatcher and(CharMatcher other) {
        return other.matches(match) ? this : NONE;
      }

      @Override public CharMatcher or(CharMatcher other) {
        return other.matches(match) ? other : super.or(other);
      }

      @Override public CharMatcher negate() {
        return isNot(match);
      }

      @GwtIncompatible("java.util.BitSet")
      @Override
      void setBits(BitSet table) {
        table.set(match);
      }
    };
  }
```

is 方法就是创建一个 FastMatcher 匿名类对象，并且实现 matches 方法，这个方法比较简单就是直接比较 char 是否相等。

replaceFrom 方法如下：

```java
public String replaceFrom(CharSequence sequence, char replacement) {
    String string = sequence.toString();
    int pos = indexIn(string);
    if (pos == -1) {
      return string;
    }
    char[] chars = string.toCharArray();
    chars[pos] = replacement;
    for (int i = pos + 1; i < chars.length; i++) {
      if (matches(chars[i])) {
        chars[i] = replacement;
      }
    }
    return new String(chars);
  }
```

比较简单不多说。

再来看端比较好的代码样式：

```java
public static CharMatcher inRange(final char startInclusive, final char endInclusive) {
    checkArgument(endInclusive >= startInclusive);
    String description = "CharMatcher.inRange('" +
        showCharacter(startInclusive) + "', '" +
        showCharacter(endInclusive) + "')";
    return inRange(startInclusive, endInclusive, description);
  }
```

不需要任何注释，只单个方法拿出来简单易懂，这就是自解释方法，方法名字，变量名字都很清楚，能明确的看到这段代码返回的 CharMatcher 匹配的是一个闭区间内的字符。

---EOF---

