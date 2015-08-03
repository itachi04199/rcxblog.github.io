---
layout: post
title: guava cache 源码学习之 Entry 包装
categories: cache
tags: java, cache
---

## 基于引用的驱逐策略

guava cache 允许设置 weak references 为 key 和 value，也允许设置 soft references 为 value。

主要有如下的方法：

```java
CacheBuilder.weakKeys()
CacheBuilder.weakValues()
CacheBuilder.softValues()
```

具体的引用区别请参考 [java 引用介绍](http://renchx.com/java-reference/)

## 分析实现

在 Segment 当中我们存放的是 ReferenceEntry 的数组：

```java
volatile AtomicReferenceArray<ReferenceEntry<K, V>> table;
```

ReferenceEntry 相比其他的 Map 的 Entry 来说，可以进行 Java 不同引用的包装。

下面一步一步分析设置过程：

```java
// CacheBuilder 当中的 weakKeys 方法
public CacheBuilder<K, V> weakKeys() {
    return setKeyStrength(Strength.WEAK);
  }


// LocalCache 当中的构造方法，如果设置了引用类型那么就根据设置的，如果没有就是强引用
keyStrength = builder.getKeyStrength();
// LocalCache 当中的构造方法，获取构造 Entry 的 Factory
entryFactory = EntryFactory.getFactory(keyStrength, usesAccessEntries(), usesWriteEntries());

// 获取方式如下，以及工厂类型，主要就是涉及到了，根据访问时间过期，或存活时间过期等
static final EntryFactory[] factories = {
      STRONG, STRONG_ACCESS, STRONG_WRITE, STRONG_ACCESS_WRITE,
      WEAK, WEAK_ACCESS, WEAK_WRITE, WEAK_ACCESS_WRITE,
    };

    static EntryFactory getFactory(Strength keyStrength, boolean usesAccessQueue,
        boolean usesWriteQueue) {
      int flags = ((keyStrength == Strength.WEAK) ? WEAK_MASK : 0)
          | (usesAccessQueue ? ACCESS_MASK : 0)
          | (usesWriteQueue ? WRITE_MASK : 0);
      return factories[flags];
    }

//CacheBuilder 当中的 getKeyStrength 方法
Strength getKeyStrength() {
    return MoreObjects.firstNonNull(keyStrength, Strength.STRONG);
  }

// Segment 当中创建 ReferenceEntry 是使用 entryFactory 来创建的
ReferenceEntry<K, V> newEntry(K key, int hash, @Nullable ReferenceEntry<K, V> next) {
      return map.entryFactory.newEntry(this, checkNotNull(key), hash, next);
    }
```

## ReferenceEntry 的数据结构

看下 ReferenceEntry 的接口方法有哪些：

```java
interface ReferenceEntry<K, V> {
    // 返回值的引用，ValueReference 也是个接口
    ValueReference<K, V> getValueReference();

    void setValueReference(ValueReference<K, V> valueReference);

    @Nullable
    ReferenceEntry<K, V> getNext();

    int getHash();

    @Nullable
    K getKey();

    long getAccessTime();

    void setAccessTime(long time);

    ReferenceEntry<K, V> getNextInAccessQueue();

    void setNextInAccessQueue(ReferenceEntry<K, V> next);

    ReferenceEntry<K, V> getPreviousInAccessQueue();

    void setPreviousInAccessQueue(ReferenceEntry<K, V> previous);

    long getWriteTime();

    void setWriteTime(long time);

    ReferenceEntry<K, V> getNextInWriteQueue();

    void setNextInWriteQueue(ReferenceEntry<K, V> next);

    ReferenceEntry<K, V> getPreviousInWriteQueue();

    void setPreviousInWriteQueue(ReferenceEntry<K, V> previous);
  }
```

下图是 ReferenceEntry 的继承体系图：

![ReferenceEntry]()

具体实现比较简单只举个简单例子：

```java
static class WeakEntry<K, V> extends WeakReference<K> implements ReferenceEntry<K, V> {
}
```

## ValueReference 结构

在 CacheBuilder 设置 value 的引用类型：

```java
public CacheBuilder<K, V> softValues() {
    return setValueStrength(Strength.SOFT);
  }

// LocalCache 的构造方法
valueStrength = builder.getValueStrength();

// Segment 当中 put 时候调用的 setValue 把传入的 value 进行了包装
void setValue(ReferenceEntry<K, V> entry, K key, V value, long now) {
      ValueReference<K, V> previous = entry.getValueReference();
      int weight = map.weigher.weigh(key, value);
      checkState(weight >= 0, "Weights must be non-negative");

      ValueReference<K, V> valueReference =
          map.valueStrength.referenceValue(this, entry, value, weight);
      entry.setValueReference(valueReference);
      recordWrite(entry, weight, now);
      previous.notifyNewValue(value);
    }
```

---EOF---
