---
layout: post
title: guava cache 源码学习之 LocalCache
categories: cache
tags: java, cache
---

##  Cache 的 maxSize

我们创建 cache 的时候可以指定这个 cache 可以最大装多少元素。下面就分析下，这个功能是如何实现的。

```java
// CacheBuilder 的使用
CacheBuilder.newBuilder().maximumSize(1000);

// LocalCache 的构造器
maxWeight = builder.getMaximumWeight();
//省略......
int segmentShift = 0;
    int segmentCount = 1;
    while (segmentCount < concurrencyLevel
           && (!evictsBySize() || segmentCount * 20 <= maxWeight)) {
      ++segmentShift;
      segmentCount <<= 1;
    }
    this.segmentShift = 32 - segmentShift;
    segmentMask = segmentCount - 1;

    this.segments = newSegmentArray(segmentCount);// segmentCount 是 Segment 数组的长度

    //省略......

    if (evictsBySize()) {// 下面这段逻辑是在创建每个 Segment 的时候把这个 Segment 能容纳的最大元素个数传递进去
      long maxSegmentWeight = maxWeight / segmentCount + 1;
      long remainder = maxWeight % segmentCount;
      for (int i = 0; i < this.segments.length; ++i) {
        if (i == remainder) {
          maxSegmentWeight--;
        }
        this.segments[i] =
            createSegment(segmentSize, maxSegmentWeight, builder.getStatsCounterSupplier().get());
      }
    } else {
      for (int i = 0; i < this.segments.length; ++i) {
        this.segments[i] =
            createSegment(segmentSize, UNSET_INT, builder.getStatsCounterSupplier().get());
      }
    }

    //getMaximumWeight 方法
    long getMaximumWeight() {
    if (expireAfterWriteNanos == 0 || expireAfterAccessNanos == 0) {
      return 0;
    }
    return (weigher == null) ? maximumSize : maximumWeight;
  }
```

因为每个 LocalCache 里面是由多个 Segment 进行切分的，所以我们在每次 put 的时候，只要当前 Segment 的元素超过了这个 maxSegmentWeight 就会移除元素。

下面来看下 Segment 当中的 put 方法：

```java
V put(K key, int hash, V value, boolean onlyIfAbsent) {
      lock();
      try {
        // 省略一些代码
        ++modCount;
        ReferenceEntry<K, V> newEntry = newEntry(key, hash, first);
        setValue(newEntry, key, value, now);// 在这个方法当中会添加个数
        table.set(index, newEntry);
        newCount = this.count + 1;
        this.count = newCount; // write-volatile
        evictEntries();
        return null;
      } finally {
        unlock();
        postWriteCleanup();
      }
    }

    void setValue(ReferenceEntry<K, V> entry, K key, V value, long now) {
      ValueReference<K, V> previous = entry.getValueReference();
      int weight = map.weigher.weigh(key, value);//默认实现都会返回1
      checkState(weight >= 0, "Weights must be non-negative");

      ValueReference<K, V> valueReference =
          map.valueStrength.referenceValue(this, entry, value, weight);
      entry.setValueReference(valueReference);
      recordWrite(entry, weight, now);//记录添加的元素
      previous.notifyNewValue(value);
    }

    void recordWrite(ReferenceEntry<K, V> entry, int weight, long now) {
      // we are already under lock, so drain the recency queue immediately
      drainRecencyQueue();
      totalWeight += weight;//给当前 Segment 添加元素个数

      if (map.recordsAccess()) {
        entry.setAccessTime(now);// 设置访问时间
      }
      if (map.recordsWrite()) {
        entry.setWriteTime(now);// 设置存活时间
      }
      accessQueue.add(entry);
      writeQueue.add(entry);
    }

    void evictEntries() {
      if (!map.evictsBySize()) {
        return;
      }

      drainRecencyQueue();
      while (totalWeight > maxSegmentWeight) {//如果当前元素个数大于最大容量，需要移除元素
        ReferenceEntry<K, V> e = getNextEvictable();
        if (!removeEntry(e, e.getHash(), RemovalCause.SIZE)) {
          throw new AssertionError();
        }
      }
    }
```

## 元素的移除策略

guava cache 提供了根据 size 进行移除元素，同时也可以根据时间来移除元素，当元素超过设定的时间后就会被移除。

- maximum 数量限制，超出时用 LRU（least-recently-used)算法清除。
- 超时限制，设置max idle time 或 max live time，在每次取元素时都会做超时检查。

看使用代码如下：

```java
CacheBuilder.newBuilder().expireAfterAccess(10, TimeUnit.MINUTES);
CacheBuilder.newBuilder().expireAfterWrite(10, TimeUnit.MINUTES);
```

一个是根据元素的闲置时间，一个是元素可存活时间。

简单说下实现：

在 Segment 当中有两个队列来记录，每次在 put 的时候会调用 preWriteCleanup 方法清除过期的元素，然后将新的元素添加到队列当中。get 的时候会把这个元素加入到 recencyQueue 这个队列当中，当下次 put 的时候会把这里面的元素重新加入到 accessQueue 当中。这样当我们每次 LRU 进行移除元素的时候，我们只需要把 accessQueue 的头元素删掉就行。

```java
    final Queue<ReferenceEntry<K, V>> writeQueue;

    final Queue<ReferenceEntry<K, V>> accessQueue;
```

下面看下具体实现：

```java
//CacheBuilder 当中的 expireAfterAccess ，会那传入的时间参数转换成纳秒
public CacheBuilder<K, V> expireAfterAccess(long duration, TimeUnit unit) {
    checkState(expireAfterAccessNanos == UNSET_INT, "expireAfterAccess was already set to %s ns",
        expireAfterAccessNanos);
    checkArgument(duration >= 0, "duration cannot be negative: %s %s", duration, unit);
    this.expireAfterAccessNanos = unit.toNanos(duration);
    return this;
  }


//LocalCache 的构造方法
expireAfterAccessNanos = builder.getExpireAfterAccessNanos();
expireAfterWriteNanos = builder.getExpireAfterWriteNanos();
```

下面看 Segment 的 put 方法：

```java
V put(K key, int hash, V value, boolean onlyIfAbsent) {
	lock();
      try {
        long now = map.ticker.read();//获取当前时间
        preWriteCleanup(now);//清除过期的元素
        //省略.....
        ++modCount;
        ReferenceEntry<K, V> newEntry = newEntry(key, hash, first);
        setValue(newEntry, key, value, now);
        table.set(index, newEntry);
        newCount = this.count + 1;
        this.count = newCount; // write-volatile
        evictEntries();
        }
        //省略.....
}

void setValue(ReferenceEntry<K, V> entry, K key, V value, long now) {
      ValueReference<K, V> previous = entry.getValueReference();
      int weight = map.weigher.weigh(key, value);
      checkState(weight >= 0, "Weights must be non-negative");

      ValueReference<K, V> valueReference =
          map.valueStrength.referenceValue(this, entry, value, weight);
      entry.setValueReference(valueReference);
      recordWrite(entry, weight, now);// 看如何设置时间
      previous.notifyNewValue(value);
    }

void recordWrite(ReferenceEntry<K, V> entry, int weight, long now) {
      // we are already under lock, so drain the recency queue immediately
      drainRecencyQueue();
      totalWeight += weight;

      if (map.recordsAccess()) {
        entry.setAccessTime(now);// 当前时间作为访问时间
      }
      if (map.recordsWrite()) {
        entry.setWriteTime(now);
      }
      accessQueue.add(entry);// 添加到队列当中
      writeQueue.add(entry);
    }

// 关键方法，每次 put 的时候都会执行这个方法，并且很多地方在处理之前都会执行这个方法，这个方法会把之前访问过的元素重新加入到 accessQueue 当中，并且 accessQueue 的 add 方法实现了，重新添加元素会添加到队列尾部。
void drainRecencyQueue() {
      ReferenceEntry<K, V> e;
      while ((e = recencyQueue.poll()) != null) {
        if (accessQueue.contains(e)) {
          accessQueue.add(e);
        }
      }
    }

// 关键的 accessQueue 当中的 add 方法，实际上是调用 offer 方法，accessQueue 内部维护了一个链表。
public boolean offer(ReferenceEntry<K, V> entry) {
      // unlink
      connectAccessOrder(entry.getPreviousInAccessQueue(), entry.getNextInAccessQueue());

      // add to tail
      connectAccessOrder(head.getPreviousInAccessQueue(), entry);
      connectAccessOrder(entry, head);

      return true;
    }

static <K, V> void connectAccessOrder(ReferenceEntry<K, V> previous, ReferenceEntry<K, V> next) {
    previous.setNextInAccessQueue(next);
    next.setPreviousInAccessQueue(previous);
  }
```

下次继续分析 guava cache 其他的设计

---EOF---
