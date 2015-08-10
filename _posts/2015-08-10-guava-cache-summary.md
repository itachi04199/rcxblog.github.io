---
layout: post
title: guava cache 源码学习总结
categories: cache
tags: java, cache
---

## 手动移除元素

- Cache.invalidate(key)
- Cache.invalidateAll(keys)
- Cache.invalidateAll()

手动移除元素源码比较简单，基本上都的调用 LocalCache 当中的 remove 方法和 clear 方法。

## Removal Listeners

在元素被移除之后我们可以进行监听。看如下使用方式：

```java
RemovalListener<String, String> ls = new RemovalListener<String, String>() {
			@Override
			public void onRemoval(RemovalNotification<String, String> notification) {
				System.out.println("key is " + notification.getKey());
				System.out.println("value is " + notification.getValue());
			}
		};

		// CacheLoader形式的Cache
		LoadingCache<String, String> cache = CacheBuilder.newBuilder().removalListener(ls).maximumSize(3).recordStats()
				.expireAfterAccess(3, TimeUnit.MINUTES).build(new CacheLoader<String, String>() {
					public String load(String key) {
						return key + "-value";
					}
				});
```

同理依旧是看 CacheBuilder 当中的实现方式：

```java
public <K1 extends K, V1 extends V> CacheBuilder<K1, V1> removalListener(
      RemovalListener<? super K1, ? super V1> listener) {
    checkState(this.removalListener == null);

    // safely limiting the kinds of caches this can produce
    @SuppressWarnings("unchecked")
    CacheBuilder<K1, V1> me = (CacheBuilder<K1, V1>) this;
    me.removalListener = checkNotNull(listener);
    return me;
  }
```

看 LocalCache 的构造方法：

```java
final Queue<RemovalNotification<K, V>> removalNotificationQueue;

removalListener = builder.getRemovalListener();
removalNotificationQueue = (removalListener == NullListener.INSTANCE)
        ? LocalCache.<RemovalNotification<K, V>>discardingQueue()
        : new ConcurrentLinkedQueue<RemovalNotification<K, V>>();
```

在 LocalCache 的构造函数当中也比较简单，至少多了一个 RemovalNotification 的队列。后面继续分析这个队列的作用。

再来看 RemovalListener 接口：

```java
public interface RemovalListener<K, V> {
  /**
   * Notifies the listener that a removal occurred at some point in the past.
   */
  // Technically should accept RemovalNotification<? extends K, ? extends V>, but because
  // RemovalNotification is guaranteed covariant, let's make users' lives simpler.
  void onRemoval(RemovalNotification<K, V> notification);
}
```

然后看 Listener 接受的参数 RemovalNotification

```java
public final class RemovalNotification<K, V> implements Entry<K, V> {
  @Nullable private final K key;
  @Nullable private final V value;
  private final RemovalCause cause;//被移除的原因

  RemovalNotification(@Nullable K key, @Nullable V value, RemovalCause cause) {
    this.key = key;
    this.value = value;
    this.cause = checkNotNull(cause);
  }

  public RemovalCause getCause() {
    return cause;
  }

  public boolean wasEvicted() {
    return cause.wasEvicted();
  }

  @Nullable @Override public K getKey() {
    return key;
  }

  @Nullable @Override public V getValue() {
    return value;
  }

  @Override public final V setValue(V value) {
    throw new UnsupportedOperationException();
  }

  @Override public boolean equals(@Nullable Object object) {
    if (object instanceof Entry) {
      Entry<?, ?> that = (Entry<?, ?>) object;
      return Objects.equal(this.getKey(), that.getKey())
          && Objects.equal(this.getValue(), that.getValue());
    }
    return false;
  }

  @Override public int hashCode() {
    K k = getKey();
    V v = getValue();
    return ((k == null) ? 0 : k.hashCode()) ^ ((v == null) ? 0 : v.hashCode());
  }

  @Override public String toString() {
    return getKey() + "=" + getValue();
  }
  private static final long serialVersionUID = 0;
}
```

当然被移除的原因有如下定义：

```java
public enum RemovalCause {
  // 用户手动清除
  EXPLICIT {
    @Override
    boolean wasEvicted() {
      return false;
    }
  },

  // 元素被替换，put 和 refresh 可能会引起这个
  REPLACED {
    @Override
    boolean wasEvicted() {
      return false;
    }
  },

  // 被 JVM GC 掉
  COLLECTED {
    @Override
    boolean wasEvicted() {
      return true;
    }
  },

  // 时间过期
  EXPIRED {
    @Override
    boolean wasEvicted() {
      return true;
    }
  },

  // 容量不够被移除
  SIZE {
    @Override
    boolean wasEvicted() {
      return true;
    }
  };

  // 是否真正被移除掉
  abstract boolean wasEvicted();
}
```

具体的移除步骤不粘贴代码了，在源码当中很容易找到。

## 总结

guava cache 值得学习的地方：

- builder 模式的使用，如果自己构造一个 cache 当然需要传递很多参数，这个构造方法我们可以看到需要传递 20 多个参数，所以封装了一个  CacheBuilder 进行构造。
- 构造 Cache 使用不上的功能，默认提供一个空实现。可以节省对象的空间。
- 活用枚举，在源码当中我们看到了大量使用枚举，然后封装抽象方法，通过枚举来实现了多种不同情况的处理。

---EOF---
