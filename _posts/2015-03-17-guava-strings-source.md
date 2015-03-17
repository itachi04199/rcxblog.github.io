---
layout: post
title: Strings 源码学习
categories: guava
tags: 笔记 java
---

首先看 3 个处理空字符串的方法：

```java
 public static String nullToEmpty(@Nullable String string) {
    return (string == null) ? "" : string;
  }

  public static @Nullable String emptyToNull(@Nullable String string) {
    return isNullOrEmpty(string) ? null : string;
  }

  public static boolean isNullOrEmpty(@Nullable String string) {
    return string == null || string.length() == 0; // string.isEmpty() in Java 6
  }
```

这三个方法比较简单，不详细解释。

然后在看两个字符串填充的方法：

```java
  public static String padStart(String string, int minLength, char padChar) {
    checkNotNull(string);  // eager for GWT.
    if (string.length() >= minLength) {
      return string;
    }
    StringBuilder sb = new StringBuilder(minLength);
    for (int i = string.length(); i < minLength; i++) {
      sb.append(padChar);
    }
    sb.append(string);
    return sb.toString();
  }

  public static String padEnd(String string, int minLength, char padChar) {
    checkNotNull(string);  // eager for GWT.
    if (string.length() >= minLength) {
      return string;
    }
    StringBuilder sb = new StringBuilder(minLength);
    sb.append(string);
    for (int i = string.length(); i < minLength; i++) {
      sb.append(padChar);
    }
    return sb.toString();
  }
```

这两个方法也比较简单，都是通过 for 循环来 append 字符。

下面是 repeat 方法的实现:

```java
  public static String repeat(String string, int count) {
    checkNotNull(string);  // eager for GWT.

    if (count <= 1) {
      checkArgument(count >= 0, "invalid count: %s", count);
      return (count == 0) ? "" : string;
    }

    // IF YOU MODIFY THE CODE HERE, you must update StringsRepeatBenchmark
    final int len = string.length();
    final long longSize = (long) len * (long) count;
    final int size = (int) longSize;
    if (size != longSize) {
      throw new ArrayIndexOutOfBoundsException("Required array size too large: "
          + String.valueOf(longSize));
    }

    final char[] array = new char[size];
    string.getChars(0, len, array, 0);
    int n;
    for (n = len; n < size - n; n <<= 1) {
      System.arraycopy(array, 0, array, n, n);
    }
    System.arraycopy(array, 0, array, n, size - n);
    return new String(array);
  }
```

方法先判断重复后的字符串是否会溢出，然后创建字符数组，先把当前字符串放到字符数组中，然后循环复制字符串的时候，复制源的长度指数增长，以最快的速度结束循环。

然后在看查找两个字符串匹配前缀和后缀的方法：

```java
public static String commonPrefix(CharSequence a, CharSequence b) {
    checkNotNull(a);
    checkNotNull(b);

    int maxPrefixLength = Math.min(a.length(), b.length());
    int p = 0;
    while (p < maxPrefixLength && a.charAt(p) == b.charAt(p)) {
      p++;
    }
    if (validSurrogatePairAt(a, p - 1) || validSurrogatePairAt(b, p - 1)) {
      p--;
    }
    return a.subSequence(0, p).toString();
  }

  public static String commonSuffix(CharSequence a, CharSequence b) {
    checkNotNull(a);
    checkNotNull(b);

    int maxSuffixLength = Math.min(a.length(), b.length());
    int s = 0;
    while (s < maxSuffixLength
        && a.charAt(a.length() - s - 1) == b.charAt(b.length() - s - 1)) {
      s++;
    }
    if (validSurrogatePairAt(a, a.length() - s - 1)
        || validSurrogatePairAt(b, b.length() - s - 1)) {
      s--;
    }
    return a.subSequence(a.length() - s, a.length()).toString();
  }
```

首先判断两个字符串中小的字符串为最大匹配长度。然后循环匹配每个字符是否相同相同的话 p 自加，最后进行截取字符串。

validSurrogatePairAt 方法来判断最后两个字符是不是合法的 “Java  平台增补字符” 。

【参考资料】

1. [Guava 是个风火轮之基础工具(3)](http://www.importnew.com/15230.html)

---EOF---


