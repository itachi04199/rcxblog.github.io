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

![ReferenceEntry](http://renchx.com/public/images/ReferenceEntry.png)

具体实现比较简单只举个简单例子：

```java
static class WeakEntry<K, V> extends WeakReference<K> implements ReferenceEntry<K, V> {
}
```

## ValueReference 结构

在 CacheBuilder 设置 value 的引用类型：

现实是 ValueReference 接口实现：

```java
/**
     * A reference to a value.
     */
    interface ValueReference<K, V> {
        /**
         * Returns the value. Does not block or throw exceptions.
         */
        @Nullable
        V get();

        /**
         * Waits for a value that may still be loading. Unlike get(), this method can block (in the case of
         * FutureValueReference).
         * 
         * @throws ExecutionException if the loading thread throws an exception
         * @throws ExecutionError if the loading thread throws an error
         */
        V waitForValue() throws ExecutionException;

        /**
         * Returns the weight of this entry. This is assumed to be static between calls to setValue.
         */
        int getWeight();

        /**
         * Returns the entry associated with this value reference, or {@code null} if this value reference is
         * independent of any entry.
         */
        @Nullable
        ReferenceEntry<K, V> getEntry();

        /**
         * 为一个指定的entry创建一个该引用的副本
         * <p>
         * {@code value} may be null only for a loading reference.
         */
        ValueReference<K, V> copyFor(ReferenceQueue<V> queue, @Nullable V value, ReferenceEntry<K, V> entry);

        /**
         * 告知一个新的值正在加载中。这个只会关联到加载值引用。
         */
        void notifyNewValue(@Nullable V newValue);

        /**
         * 当一个新的value正在被加载的时候，返回true。不管是否已经有存在的值。这里加锁方法返回的值对于给定的ValueReference实例来说是常量。
         * 
         */
        boolean isLoading();

        /**
         * 返回true，如果该reference包含一个活跃的值,意味着在cache里仍然有一个值存在。活跃的值包含：cache查找返回的，等待被移除的要被驱赶的值； 非激活的包含：正在加载的值，
         */
        boolean isActive();
    }
```

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
