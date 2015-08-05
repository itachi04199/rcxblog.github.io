---
layout: post
title: guava cache 源码学习之 CacheStats
categories: cache
tags: java, cache
---

## CacheStats

我们已经知道了如何创建缓存，我们也想要收集和统计缓存是如何被执行和使用的。有一个非常简单的方式获取缓存执行的信息。但是跟踪缓存操作会降低性能。如果想要收集缓存的信息，我们只需要在创建缓存的时候特殊说明 `recordStats()`。

```java
public void guavaTest() throws Exception {
		// CacheLoader形式的Cache
		LoadingCache<String, String> cache = CacheBuilder.newBuilder().maximumSize(10).recordStats()
				.expireAfterAccess(5, TimeUnit.MINUTES).build(new CacheLoader<String, String>() {
					public String load(String key) {
						return key + "-value";
					}
				});
		cache.put("rcx1", "va1");
		cache.put("rcx2", "va2");
		cache.put("rcx3", "va3");
		cache.getIfPresent("rcx1");
		cache.put("rcx4", "rcx4");
		cache.put("rcx5", "rcx5");
		cache.put("rcx6", "rcx6");

		cache.get("rcx1");
		cache.get("rcx2");
		cache.get("rcx3");
		cache.get("rcx7");

		CacheStats stats = cache.stats();
		System.out.println(stats.hitCount());
		System.out.println(stats.hitRate());
	}
```

依然分析下源码：

首先看一下返回的 CacheStats 结构：

```java

  private final long hitCount;
  private final long missCount;
  private final long loadSuccessCount;
  private final long loadExceptionCount;
  private final long totalLoadTime;
  private final long evictionCount;

  //构造函数，命中次数等都是构造的时候传入进来的。
  public CacheStats(long hitCount, long missCount, long loadSuccessCount,
      long loadExceptionCount, long totalLoadTime, long evictionCount) {
    checkArgument(hitCount >= 0);
    checkArgument(missCount >= 0);
    checkArgument(loadSuccessCount >= 0);
    checkArgument(loadExceptionCount >= 0);
    checkArgument(totalLoadTime >= 0);
    checkArgument(evictionCount >= 0);

    this.hitCount = hitCount;
    this.missCount = missCount;
    this.loadSuccessCount = loadSuccessCount;
    this.loadExceptionCount = loadExceptionCount;
    this.totalLoadTime = totalLoadTime;
    this.evictionCount = evictionCount;
  }

  public long requestCount() {
    return hitCount + missCount;
  }

  public long hitCount() {
    return hitCount;
  }

  public double hitRate() {
    long requestCount = requestCount();
    return (requestCount == 0) ? 1.0 : (double) hitCount / requestCount;
  }

  public long missCount() {
    return missCount;
  }

  public double missRate() {
    long requestCount = requestCount();
    return (requestCount == 0) ? 0.0 : (double) missCount / requestCount;
  }

  public long loadCount() {
    return loadSuccessCount + loadExceptionCount;
  }

  public long loadSuccessCount() {
    return loadSuccessCount;
  }

  public long loadExceptionCount() {
    return loadExceptionCount;
  }

  public double loadExceptionRate() {
    long totalLoadCount = loadSuccessCount + loadExceptionCount;
    return (totalLoadCount == 0)
        ? 0.0
        : (double) loadExceptionCount / totalLoadCount;
  }

  public long totalLoadTime() {
    return totalLoadTime;
  }

  public double averageLoadPenalty() {
    long totalLoadCount = loadSuccessCount + loadExceptionCount;
    return (totalLoadCount == 0)
        ? 0.0
        : (double) totalLoadTime / totalLoadCount;
  }

  public long evictionCount() {
    return evictionCount;
  }

  public CacheStats minus(CacheStats other) {
    return new CacheStats(
        Math.max(0, hitCount - other.hitCount),
        Math.max(0, missCount - other.missCount),
        Math.max(0, loadSuccessCount - other.loadSuccessCount),
        Math.max(0, loadExceptionCount - other.loadExceptionCount),
        Math.max(0, totalLoadTime - other.totalLoadTime),
        Math.max(0, evictionCount - other.evictionCount));
  }

  public CacheStats plus(CacheStats other) {
    return new CacheStats(
        hitCount + other.hitCount,
        missCount + other.missCount,
        loadSuccessCount + other.loadSuccessCount,
        loadExceptionCount + other.loadExceptionCount,
        totalLoadTime + other.totalLoadTime,
        evictionCount + other.evictionCount);
  }

  @Override
  public int hashCode() {
    return Objects.hashCode(hitCount, missCount, loadSuccessCount, loadExceptionCount,
        totalLoadTime, evictionCount);
  }

  @Override
  public boolean equals(@Nullable Object object) {
    if (object instanceof CacheStats) {
      CacheStats other = (CacheStats) object;
      return hitCount == other.hitCount
          && missCount == other.missCount
          && loadSuccessCount == other.loadSuccessCount
          && loadExceptionCount == other.loadExceptionCount
          && totalLoadTime == other.totalLoadTime
          && evictionCount == other.evictionCount;
    }
    return false;
  }

  @Override
  public String toString() {
    return MoreObjects.toStringHelper(this)
        .add("hitCount", hitCount)
        .add("missCount", missCount)
        .add("loadSuccessCount", loadSuccessCount)
        .add("loadExceptionCount", loadExceptionCount)
        .add("totalLoadTime", totalLoadTime)
        .add("evictionCount", evictionCount)
        .toString();
  }
}

```

下面看下 CacheBuilder 我们进行 recordStats 方法的时候操作：

```java
//CacheBuilder 当中，会把 statsCounterSupplier 进行赋值，这当然是为了在 LoaclCache 当中使用
public CacheBuilder<K, V> recordStats() {
    statsCounterSupplier = CACHE_STATS_COUNTER;
    return this;
  }
static final Supplier<StatsCounter> CACHE_STATS_COUNTER =
      new Supplier<StatsCounter>() {
    @Override
    public StatsCounter get() {
      return new SimpleStatsCounter();
    }
  };

// LocalCache 成员变量定义
final StatsCounter globalStatsCounter;
// LocalCache 构造方法
globalStatsCounter = builder.getStatsCounterSupplier().get();
```

我们先看 StatsCounter 接口的方法：

```java
public interface StatsCounter {
    void recordHits(int count);
    void recordMisses(int count);
    void recordLoadSuccess(long loadTime);
    void recordLoadException(long loadTime);
    void recordEviction();
	//这个方法是返回当前缓存状态的快照，也就是说不同时候获取缓存的快照都是不同的实例
    CacheStats snapshot();
  }
```

下面是简单的 SimpleStatsCounter 的实现：

```java
public static final class SimpleStatsCounter implements StatsCounter {
    private final LongAddable hitCount = LongAddables.create();
    private final LongAddable missCount = LongAddables.create();
    private final LongAddable loadSuccessCount = LongAddables.create();
    private final LongAddable loadExceptionCount = LongAddables.create();
    private final LongAddable totalLoadTime = LongAddables.create();
    private final LongAddable evictionCount = LongAddables.create();

    public SimpleStatsCounter() {}

    @Override
    public void recordHits(int count) {
      hitCount.add(count);
    }

    @Override
    public void recordMisses(int count) {
      missCount.add(count);
    }

    @Override
    public void recordLoadSuccess(long loadTime) {
      loadSuccessCount.increment();
      totalLoadTime.add(loadTime);
    }

    @Override
    public void recordLoadException(long loadTime) {
      loadExceptionCount.increment();
      totalLoadTime.add(loadTime);
    }

    @Override
    public void recordEviction() {
      evictionCount.increment();
    }

    @Override
    public CacheStats snapshot() {
      return new CacheStats(
          hitCount.sum(),
          missCount.sum(),
          loadSuccessCount.sum(),
          loadExceptionCount.sum(),
          totalLoadTime.sum(),
          evictionCount.sum());
    }

    public void incrementBy(StatsCounter other) {
      CacheStats otherStats = other.snapshot();
      hitCount.add(otherStats.hitCount());
      missCount.add(otherStats.missCount());
      loadSuccessCount.add(otherStats.loadSuccessCount());
      loadExceptionCount.add(otherStats.loadExceptionCount());
      totalLoadTime.add(otherStats.totalLoadTime());
      evictionCount.add(otherStats.evictionCount());
    }
  }
```

里面主要是简单地缓存累加，此外由于多线程下 Long 类型的线程非安全性，所以也进行了一下封装。

当然在 get 的时候如果我们在缓存当中获取到了数据就会命中加一，没获取到就 miss 加一。

## CacheBuilder 初始化默认值

CacheBuilder 当中可以看到这些默认的初始化，有两套引用：Null 对象和 Empty 对象，显然 Null 会更省空间，但我们在创建的时候将决定不使用某特性的时候，就会使用Null来创建，否则使用 Empty 来完成初始化工作。

例如，如果我们不调用方法 recordStats，那么默认的 StatsCounter 就是下面的。

```java
static final Supplier<? extends StatsCounter> NULL_STATS_COUNTER = Suppliers.ofInstance(
      new StatsCounter() {
        @Override
        public void recordHits(int count) {}

        @Override
        public void recordMisses(int count) {}

        @Override
        public void recordLoadSuccess(long loadTime) {}

        @Override
        public void recordLoadException(long loadTime) {}

        @Override
        public void recordEviction() {}

        @Override
        public CacheStats snapshot() {
          return EMPTY_STATS;
        }
      });
static final CacheStats EMPTY_STATS = new CacheStats(0, 0, 0, 0, 0, 0);
```

当然还有其他的属性设置，例如 RemovalListener，Ticker 等。

---EOF---
