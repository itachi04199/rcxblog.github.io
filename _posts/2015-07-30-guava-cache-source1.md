---
layout: post
title: guava cache 源码学习1
categories: cache
tags: java, cache
---

## 使用场景

在 guava wiki 当中描述了适合使用 guava cache 的一些场景：

* 愿意消耗一些内存空间来提升速度；

* 能够预计某些key会被查询一次以上；

* 缓存中存放的数据总量不会超出内存容量(Guava Cache 是单个应用运行时的本地缓存)。

## 用法

下面是个 guava cache 的简单用法:

```java
public void guavaTest() throws Exception {
		// CacheLoader形式的Cache
		LoadingCache<String, String> cache = CacheBuilder.newBuilder().maximumSize(1000)
				.expireAfterWrite(10, TimeUnit.SECONDS).build(new CacheLoader<String, String>() {
					public String load(String key) {
						return key + "-value";
					}
				});

		// Callable形式的Cache
		Cache<String, String> cache2 = CacheBuilder.newBuilder().maximumSize(1000)
				.expireAfterWrite(10, TimeUnit.SECONDS).build();
		cache2.get("rcx1", new Callable<String>() {
			@Override
			public String call() throws Exception {
				return "value";
			}
		});
	}
```

## Cache 接口

看下 guava cache 定义的 Cache 接口，有哪些方法：

```java
public interface Cache<K, V> {

  @Nullable//如果key存在返回value，不存在返回null，不会触发值的加载
  V getIfPresent(Object key);

  V get(K key, Callable<? extends V> valueLoader) throws ExecutionException;

  ImmutableMap<K, V> getAllPresent(Iterable<?> keys);

  void put(K key, V value);

  void putAll(Map<? extends K,? extends V> m);

  void invalidate(Object key);

  void invalidateAll(Iterable<?> keys);

  void invalidateAll();

  long size();

  CacheStats stats();

  ConcurrentMap<K, V> asMap();

  void cleanUp();
}
```

## CacheBuilder

Builder 模式，构造一个 Cache 需要设置许多参数，所以 guava 使用了 Builder 模式，来创建 Cache，每次设置好一个参数后都会返回自身的对象实例，这样可以链式的调用下去。

下面是 CacheBuilder 可以设置的参数：

```java
public CacheBuilder<K, V> maximumSize(long size) {}
public CacheBuilder<K, V> expireAfterAccess(long duration, TimeUnit unit) {}
public CacheBuilder<K, V> expireAfterWrite(long duration, TimeUnit unit) {}
public CacheBuilder<K, V> concurrencyLevel(int concurrencyLevel) {}
public CacheBuilder<K, V> maximumWeight(long weight) {}
public CacheBuilder<K, V> weakKeys() {}

.......等
```

这些属性有什么作用后续慢慢分析。

## 浅析 LocalCache

LocalCache 可以看做是 guava cache 当中最核心的一个类，因为这个类的实现基本上跟 ConcurrentHashMap 的实现方式基本类似。

关于 ConcurrentHashMap 有如下推荐阅读：

[ConcurrentHashMap阅读1](http://www.ibm.com/developerworks/cn/java/java-lo-concurrenthashmap/)

[ConcurrentHashMap阅读2](http://www.infoq.com/cn/articles/ConcurrentHashMap)

[ConcurrentHashMap阅读3](http://jeffzzf.github.io/2015/05/29/Java-ConcurrentHashMap%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90/)

简单看下 LocalCache 类的声明：

```java
class LocalCache<K, V> extends AbstractMap<K, V> implements ConcurrentMap<K, V> {
	LocalCache(
      CacheBuilder<? super K, ? super V> builder, @Nullable CacheLoader<? super K, V> loader) {
    concurrencyLevel = Math.min(builder.getConcurrencyLevel(), MAX_SEGMENTS);

    keyStrength = builder.getKeyStrength();
    valueStrength = builder.getValueStrength();

    keyEquivalence = builder.getKeyEquivalence();
    valueEquivalence = builder.getValueEquivalence();

    maxWeight = builder.getMaximumWeight();
    weigher = builder.getWeigher();
    expireAfterAccessNanos = builder.getExpireAfterAccessNanos();
    expireAfterWriteNanos = builder.getExpireAfterWriteNanos();
    refreshNanos = builder.getRefreshNanos();

    removalListener = builder.getRemovalListener();
    removalNotificationQueue = (removalListener == NullListener.INSTANCE)
        ? LocalCache.<RemovalNotification<K, V>>discardingQueue()
        : new ConcurrentLinkedQueue<RemovalNotification<K, V>>();

    ticker = builder.getTicker(recordsTime());
    entryFactory = EntryFactory.getFactory(keyStrength, usesAccessEntries(), usesWriteEntries());
    globalStatsCounter = builder.getStatsCounterSupplier().get();
    defaultLoader = loader;

    int initialCapacity = Math.min(builder.getInitialCapacity(), MAXIMUM_CAPACITY);
    // ...... 省略
  }
}
```

从上面的构造方法，我们可以看到是直接从 CacheBuilder 当中获取参数，设置到 LocalCache 当中。

## 两种类型的 Cache

在用法当中我们演示了 guava cache 的两种创建方式：

- CacheLoader 形式的 Cache
- Callable 形式的 Cache

下面看下 CacheBuilder.build 的时候这2种情况的调用有什么不同：

```java
//CacheLoader 形式的 Cache
public <K1 extends K, V1 extends V> LoadingCache<K1, V1> build(
      CacheLoader<? super K1, V1> loader) {
    checkWeightWithWeigher();
    return new LocalCache.LocalLoadingCache<K1, V1>(this, loader);
  }

//Callable 形式的 Cache
public <K1 extends K, V1 extends V> Cache<K1, V1> build() {
    checkWeightWithWeigher();
    checkNonLoadingCache();
    return new LocalCache.LocalManualCache<K1, V1>(this);
  }
```

从上面代码可以看出来都是创建的 LocalCache 当中的静态内部类实例，只是这2个内部类不同。

下面看下 LocalManualCache 类的定义：

```java
static class LocalManualCache<K, V> implements Cache<K, V>, Serializable {
    final LocalCache<K, V> localCache;

    LocalManualCache(CacheBuilder<? super K, ? super V> builder) {
      this(new LocalCache<K, V>(builder, null));
    }

    private LocalManualCache(LocalCache<K, V> localCache) {
      this.localCache = localCache;
    }

    public V getIfPresent(Object key) {
      return localCache.getIfPresent(key);
    }

    public V get(K key, final Callable<? extends V> valueLoader) throws ExecutionException {
      checkNotNull(valueLoader);
      return localCache.get(key, new CacheLoader<Object, V>() {
        @Override
        public V load(Object key) throws Exception {
          return valueLoader.call();
        }
      });
    }
    ...... 省略
}
```

从代码实现看出实际上是一个 adapter 组件，并且绝大部分实现都是直接调用 LocalCache 的方法，或者加一些参数判断和聚合。在它核心的构造函数中，就是直接调用 LocalCache 构造函数，对于 loader 对象直接设null值。

下面看下 LocalLoadingCache 类的定义：

```java
static class LocalLoadingCache<K, V>
      extends LocalManualCache<K, V> implements LoadingCache<K, V> {

    LocalLoadingCache(CacheBuilder<? super K, ? super V> builder,
        CacheLoader<? super K, V> loader) {
      super(new LocalCache<K, V>(builder, checkNotNull(loader)));
    }

	@Override
    public V get(K key) throws ExecutionException {
      return localCache.getOrLoad(key);
    }
    ...... 省略
}
```

LocalLoadingCache 主要对 get 方法进行了重写，调用的还是 LocalCache 的方法。

本次先进行简单的分析，后续进行全面的分析。

【参考资料】

1. [Guava LocalCache 缓存介绍及实现源码深入剖析](http://ketao1989.github.io/posts/Guava-Cache-Guide-And-Implement-Analyse.html)

---EOF---
