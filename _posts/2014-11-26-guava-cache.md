---
layout: post
title: Guava缓存
categories: guava
tags: 笔记 java
---

#### MapMaker

MapMaker 使用流 api，允许我们快速创建 ConcurrentHashMap，让我们看下面的例子：

```java
 ConcurrentMap<String,Book> books = new MapMaker().concurrencyLevel(2).
 					softValues().makeMap();
```

我们创建了一个 String 为 key，Book 为值的 ConcurrentHashMap。concurrencyLevel 方法可以设置同时修改的并发量。

softValues 方法设置成 map 里面的值包装成软引用，并且当 gc 的时候有可能被回收。

其他的方法也有 weakKeys 或者 weakValues，但是没有 softKeys 方法。

#### Guava caches

Guava cache 有两个基本的接口，Cache 和 LoadingCache。LoadingCache 继承于 Cache 接口。

##### Cache

Cache 接口提供从 key 到 value 的映射，cache 中的一些方法在基本的 HashMap 当中已经提供。

在传统使用 map 或者 cache，我们出示一个 key，如果缓存中对应这个 key 有 value 就返回这个 value，如果没有我们会返回 null。如果在缓存中放置一个值，我们会调用如下这样的方法：

```java
put(key,value);
```

在这里我们明确的把 key 对应的 value 放置到 cache 或 map 中。

Guava 当中的 Cache 接口有传统的 put 方法，但是看下面 Cache 接口当中自加载风格的方法:

```java
V value = cache.get(key, Callable<? Extends V> value);
```

上面的方法如果取回了值会返回，如果没取到它将会从 Callable 实例当中获取到值并且与这个 key 关联，并且返回这个值。他相当于下面的代码：

```java
   value = cache.get(key);
   if(value == null){
       value = someService.retrieveValue();
       cache.put(key,value);
   }
```
使用 Callable 对象意味着异步操作可能存在。但是我们如果不想执行异步任务的话，我们可以使用 Callables 类。Callables 类有一个方法如下：

```java
	Callable<String> value = Callables.returning("Foo");
```

上面的代码，returning 方法会构建并且返回一个 Callable 实例。所以我们可以重新实现我们前面的例子：

```java
cache.get(key,Callables.returning(someService.retrieveValue());
```

记住如果 key 有值则缓存就会返回。如果我们想 key 不存在的时候返回 null，我们可以使用 getIfPresent 方法。有一些方法可以使在缓存中的值无效。如下

- invalidate(key):方法使key对应的值被丢弃
- invalidateAll():方法丢弃缓存中的所有值
- invalidateAll(Iterable<?> keys):方法丢弃所有给定key对应的值

#### LoadingCache

LoadingCache 接口继承于 Cache 接口并且自带自加载功能。看下面的例子：

```java
Book book = loadingCache.get(id);
```

如果执行 get 方法的时候 book 不存在，那么 LoadingCache 将会加载对象，并且将它存储在缓存中，并且返回这个值。

##### Loading values

LoadingCache 是线程安全的，使用相同的 key 去调用 get 方法，在加载的时候会被阻塞。一旦值被加载完毕，将会返回这个值，其他的加载会被转为原始的 get 方法调用。然而，不同的 key 一起调用 get 方法，则会并发处理。如果我们有一个 key 的集合，并且我们想检索每一个 key 的 value，我们可以使用如下方法：

```java
ImmutableMap<key,value> map = cache.getAll(Iterable<? Extends key>);
```

可以看到，getAll 方法返回 ImmutableMap 。返回的值里面有可能全部都在缓存中，有可能都是新检索的，也有可能一部分是缓存中的一部分是新检索的。

#### 缓存中刷新值

LoadingCache 也提供了一个机制来刷新缓存中的值：

```java
refresh(key);
```

调用 refresh 方法，LoadingCache 将为这个 key 检索一个新的值。当前的值直到新的值返回才会被丢弃。这就意味着在刷新的过程当中调用 get 方法，将会返回缓存中的值。如果在刷新的时候抛出了异常，原始在缓存中的值将不变。如果检索值是异步的，可能返回的值是刷新之后的。

#### CacheBuilder

CacheBuilder 提供了获得 Cache 和 LoadingCache 实例的方式。Cache 上有很多选项可以指定，看下面的例子如何指定在缓存中失效的：

```java
LoadingCache<String,TradeAccount> tradeAccountCache = CacheBuilder.newBuilder()
					.expireAfterWrite(5L, TimeUnit.Minutes)
					.maximumSize(5000L)
					.removalListener(new TradeAccountRemovalListener())
					.ticker(Ticker.systemTicker())
					.build(new CacheLoader<String, TradeAccount>() {
                    @Override
                    public TradeAccount load(String key) throws Exception {
                    return tradeAccountService.getTradeAccountById(key);
					}
				});

public class TradeAccount {
private String id;
private String owner;
private double balance;
}
```

看下上面的例子：

1. 首先，我们调用 expireAfterWrite 方法，它将自动从缓存中移除超过5分钟的元素。
2. 我们可以使用 maximumSize 方法指定缓存中的最大数目。最少使用的元素将会被移除当缓存的大小接近最大数目的时候，并不一定等于或超过最大数目。
3. 我们添加一个 RemovalListener 将会收到一个元素被移除的通知。
4. 我们通过 ticker 方法添加一个 Ticker 实例，支持纳秒级别的缓存元素过期。
5. 调用 build 方法，传入一个 CacheLoader 实例， 当对应的 key 不存在 value 的时候，进行加载元素。

在我们的例子中，我们可以看到如何将缓存元素过期，在元素最后使用时间超过缓存时间的时候。

```java
LoadingCache<String,Book> bookCache = CacheBuilder.newBuilder()
					.expireAfterAccess(20L,TimeUnit.MINUTES)
					.softValues()
					.removalListener(new BookRemovalListener())
					.build(new CacheLoader<String, Book>() {
                    @Override
                    public Book load(String key) throws Exception
                    {
                    	return bookService.getBookByIsbn(key);
                    }
                    });
```

上面的例子与前一个例子有些不同：

1. 我们通过 expireAfterAccess 方法，指定元素在最后访问过时间超过20分钟后过期。
2. 代替指定明确的缓存大小，我们 JVM 来限制缓存的大小，通过 softValues() 方法来使用 SoftReferences 包装缓存中的值。当内存不够时，缓存中的元素会被移除。SoftReferences 被 gc 回收的算法是基于 LRU (最新最少使用) 算法。

下面看最后的一个例子将展示缓存当中如何自动刷新值：

```java
LoadingCache<String,TradeAccount> tradeAccountCache = CacheBuilder.newBuilder()
										.concurrencyLevel(10)
										.refreshAfterWrite(5L,TimeUnit.SECONDS)
										.ticker(Ticker.systemTicker())
										.build(new CacheLoader<String, TradeAccount>() {
                                        @Override
                                        public TradeAccount load(String key) throws Exception {
                                        return tradeAccountService.getTradeAccountById(key);
                                        }
                                        });
```

上面的例子有一些改变:

1. 我们通过 concurrencyLevel 方法指定并发修改量，如果不指定默认是4。
2. 在过期后我们使用刷新值来代替移除值。注意触发刷新值的关键是，请求这个值并且这个值已经过期。
3. 我们添加一个 ticker 来控制哪些值需要刷新。

#### CacheBuilderSpec

CacheBuilderSpec 类可以创建 CacheBuilder 实例通过解析字符串来对 CacheBuilder 进行设置，下面是一个例子：

```java
String spec = "concurrencyLevel=10,expireAfterAccess=5m,softValues";
CacheBuilderSpec cacheBuilderSpec = CacheBuilderSpec.parse(spec);
CacheBuilder cacheBuilder = CacheBuilder.from(cacheBuilderSpec);
cacheBuilder.ticker(Ticker.systemTicker())
			.removalListener(new TradeAccountRemovalListener())
            .build(new CacheLoader<String, TradeAccount>() {
                   @Override
                   public TradeAccount load(String key) throws Exception {
                       return tradeAccountService.getTradeAccountById(key);
			} });
```

我们会创建与上面最后例子相同的 CacheBuilder 的实例。通常字符串形式的创建 CacheBuilder 会在命令行或者从属性文件总解析。

#### CacheLoader

CacheLoader 是一个抽象类，因为它的 load 方法上抽象的。也有一个 loadAll 方法，它接受一个 Iterable 对象，但是 loadAll 方法实际上是调用的 load 方法。CacheLoader 有两个静态方法允许我们利用函数编程结构。第一个方法如下：

```java
CacheLoader<Key,value> cacheLoader = CacheLoader.from(Function<Key,Value> func);
```

在这我们传入一个 Function 对象将会转换输入对象到输出对象。CacheLoader.from 方法将会返回一个 CacheLoader 实例，这个CacheLoader的 key 将会作为输入对象，并且调用 Function 里面的 apply 方法转换出来的对象当做输出对象。

第二个方法如下：

```java
   CacheLoader<Object,Value> cacheLoader =  CacheLoader.from(Supplier<Value> supplier);
```

这个方法我们传入一个 Supplier 实例。当缓存中对应 key 的值不存在的时候 CacheLoader 将会返回 Supplier.get() 方法执行的结果。

#### CacheStats

我们已经知道了如何创建缓存，我们也想要收集和统计缓存是如何被执行和使用的。有一个非常简单的方式获取缓存执行的信息。但是跟踪缓存操作会降低性能。如果想要收集缓存的信息，我们只需要在创建缓存的时候特殊说明。

```java
LoadingCache<String,TradeAccount> tradeAccountCache = CacheBuilder.newBuilder().recordStats()
```

我们用熟悉的方式来创建 LoadingCache 实例，如果收集缓存的信息我们只需要调用 recordStats 方法。在缓存实例上调用 stats 方法会返回 CacheStats 对象，里面有缓存的收集信息。

下面上可以从 CacheStats 当中可以获取的信息：
- 加载新值的平均时间
- 击中缓存的概率
- 缓存 miss 的概率

#### RemovalListener

在前面的 CacheBuilder 的例子当中我们看到了如何添加一个 RemovalListener 实例到我们的缓存当中。RemovalListener 是一个接口并且有一个方法 onRemoval，并且接受一个 RemovalNotification 对象参数。如下：

```java
RemovalListener<K,V>
```

K 是我们想要监听的 key 的类型， V 是 value 的类型。如果我们想知道任何类型元素被移除，我们可以简单使用 Object 类型放在 K 和 V 上。

#### RemovalNotification

RemovalNotification 是接受 RemovalListener 移除元素信号的实例。RemovalNotification 实现了 Map.Entry 接口，并且我们可以访问在缓存当中的真实 key 和 value。需要注意的是这些值有可能是 null 如果他们已经被垃圾回收掉了。

我们也可以调用 RemovalNotification 实例的 getCause() 方法来获取元素移除的原因，它将返回 RemovalCause 的枚举。枚举的值有下面几种可能：

- COLLECTED: 这个表示 key 或者 value 当中的一个被gc掉了
- EXPIRED: 表示元素已经过期
- EXPLICIT: 用户手动移除
- REPLACED: 元素没被移除，但是值已经被替换掉
- SIZE: 元素被移除是因为缓存的大小不够

#### RemovalListeners

RemovalListeners 类让我们异步处理移除元素的通知更加简单。我们可以简单是使用 RemovalListeners.asynchronous 方法：

```java
RemovalListener<String,TradeAccount> myRemovalListener = new
RemovalListener<String, TradeAccount>() {
    @Override
    public void onRemoval(RemovalNotification<String,
    TradeAccount> notification) {
    //Do something here
    }
};
RemovalListener<String,TradeAccount> removalListener =
RemovalListeners.asynchronous(myRemovalListener,executorService);
```

我们给 asynchronous 方法传递 RemovalListener 实例和 ExecutorService 实例，并且返回一个 RemovalListener 实例，它会异步的处理移除元素的通知。

---EOF---

