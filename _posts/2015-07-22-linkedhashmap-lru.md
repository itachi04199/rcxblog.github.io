---
layout: post
title: 使用 LinkedHashMap 实现 LRU 算法
categories: cache
tags: java, cache
---

## LinkedHashMap 源码分析

```java
public class LinkedHashMap<K,V>
    extends HashMap<K,V>
    implements Map<K,V>
```

上面是 LinkedHashMap 的继承结构。然后看下构造方法：

```java
private transient Entry<K,V> header;//内部链表的头
private final boolean accessOrder;//是否按照访问的顺序排序，默认是 false
public LinkedHashMap(int initialCapacity, float loadFactor) {
        super(initialCapacity, loadFactor);
        accessOrder = false;
    }
    public LinkedHashMap(int initialCapacity) {
        super(initialCapacity);
        accessOrder = false;
    }
    public LinkedHashMap() {
        super();
        accessOrder = false;
    }
    public LinkedHashMap(int initialCapacity,
                         float loadFactor,
                         boolean accessOrder) {
        super(initialCapacity, loadFactor);
        this.accessOrder = accessOrder;
    }
```

构造方法比较简单，主要是多个两个成员变量，并且构造方法当中多了 accessOrder。

下面看下 put 方法，LinkedHashMap 并没有重新实现 put 方法，但是重写了 HashMap 当中的几个 put 调用的方法：

下面是 HashMap 的 put 方法：

```java
public V put(K key, V value) {
        if (table == EMPTY_TABLE) {
            inflateTable(threshold);
        }
        if (key == null)
            return putForNullKey(value);
        int hash = hash(key);
        int i = indexFor(hash, table.length);
        for (Entry<K,V> e = table[i]; e != null; e = e.next) {
            Object k;
            if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
                V oldValue = e.value;
                e.value = value;
                e.recordAccess(this);
                return oldValue;
            }
        }

        modCount++;
        addEntry(hash, key, value, i);
        return null;
    }
```

LinkedHashMap 当中重写了 addEntry 方法和 createEntry 方法。下面看下：

```java
void addEntry(int hash, K key, V value, int bucketIndex) {
        super.addEntry(hash, key, value, bucketIndex);

        // Remove eldest entry if instructed
        Entry<K,V> eldest = header.after;
        if (removeEldestEntry(eldest)) {
            removeEntryForKey(eldest.key);
        }
    }

// HashMap 当中的 addEntry 如下
void addEntry(int hash, K key, V value, int bucketIndex) {
        if ((size >= threshold) && (null != table[bucketIndex])) {
            resize(2 * table.length);
            hash = (null != key) ? hash(key) : 0;
            bucketIndex = indexFor(hash, table.length);
        }

        createEntry(hash, key, value, bucketIndex);
    }
```

下面的 createEntry 方法：

```java
void createEntry(int hash, K key, V value, int bucketIndex) {
        HashMap.Entry<K,V> old = table[bucketIndex];
        Entry<K,V> e = new Entry<>(hash, key, value, old);
        table[bucketIndex] = e;
        e.addBefore(header);
        size++;
    }
```

从上面这两个重写的方法来看，在 LinkedHashMap 当 put 一组 key - value 的时候，我们会在 HashMap 的结构当中添加一个元素，并且会在自己的特有的链表当中添加一个元素。

看下 LinkedHashMap 内部的 Entry 实现代码：

```java
private static class Entry<K,V> extends HashMap.Entry<K,V> {
        Entry<K,V> before, after;

        Entry(int hash, K key, V value, HashMap.Entry<K,V> next) {
            super(hash, key, value, next);
        }

        private void remove() {
            before.after = after;
            after.before = before;
        }

        private void addBefore(Entry<K,V> existingEntry) {
            after  = existingEntry;
            before = existingEntry.before;
            before.after = this;
            after.before = this;
        }

        void recordAccess(HashMap<K,V> m) {
            LinkedHashMap<K,V> lm = (LinkedHashMap<K,V>)m;
            if (lm.accessOrder) {
                lm.modCount++;
                remove();
                addBefore(lm.header);
            }
        }

        void recordRemoval(HashMap<K,V> m) {
            remove();
        }
    }
```

这个结构就是一个双向循环列表，所以 LinkedHashMap 的内部数据结构就是如下图：

![LinkedHashMap数据结构]()

## 实现 LRU 算法

LRU 算法就是近期最少使用算法。当我们要用 LinkedHashMap 来实现的时候，其实我们就是用他内部的双向链表，每次 put 的时候我们把这个元素加入到链表尾部，然后 get 的时候也会把元素重新添加到尾部，这样就简单的描述了一个 LRU 算法。

那么实现代码如下：

```java
package com.rcx.test.cache;

import java.util.LinkedHashMap;
import java.util.Map;

public class LRULinkedHashMap<K, V> extends LinkedHashMap<K, V> {

	private static final long serialVersionUID = 6802031150943192407L;
	private int capacity;

	LRULinkedHashMap(int capacity) {
		super(16, 0.75f, true);
		this.capacity = capacity;
	}

	@Override
	public boolean removeEldestEntry(Map.Entry<K, V> eldest) {
		return size() > capacity;
	}

	public static void main(String[] args) throws Exception {
		LRULinkedHashMap<String, String> map = new LRULinkedHashMap<String, String>(4);

		map.put("rcx1", "a1");
		map.put("rcx2", "a2");
		map.put("rcx3", "a3");
		map.put("rcx4", "a4");
		map.put("rcx5", "a5");

		System.out.println(map.get("rcx1"));
	}
}
```

看代码实现的很简单，只需要重写一个 removeEldestEntry 就行，这个方法只需要判断当前的 size 是不是大于最大容量，那么我们重新看下这是为什么：

首先我们创建的 LinkedHashMap 第 3 个参数传的是 true 。这样 accessOrder 就是 addEntry 了，那么就重新看下 put 方法。

```java
void addEntry(int hash, K key, V value, int bucketIndex) {
        super.addEntry(hash, key, value, bucketIndex);

        Entry<K,V> eldest = header.after;//header 后面的元素是最不常用的元素
        // removeEldestEntry 这个方法我们重写了，大于最大容量就进入 if 语句
        if (removeEldestEntry(eldest)) {
            removeEntryForKey(eldest.key);
        }
    }

// HashMap 当中的 removeEntryForKey
final Entry<K,V> removeEntryForKey(Object key) {
        if (size == 0) {
            return null;
        }
        int hash = (key == null) ? 0 : hash(key);
        int i = indexFor(hash, table.length);
        Entry<K,V> prev = table[i];
        Entry<K,V> e = prev;

        while (e != null) {
            Entry<K,V> next = e.next;
            Object k;
            if (e.hash == hash &&
                ((k = e.key) == key || (key != null && key.equals(k)))) {
                modCount++;
                size--;
                if (prev == e)
                    table[i] = next;
                else
                    prev.next = next;
                e.recordRemoval(this);// 会调用 Entry 的这个方法，并且 LinkedHashMap.Entry 重写了这个方法
                return e;
            }
            prev = e;
            e = next;
        }

        return e;
    }
```

我们看下 LinkedHashMap.Entry 的 recordRemoval 实现：

```java
void recordRemoval(HashMap<K,V> m) {
            remove();
        }
private void remove() {
            before.after = after;
            after.before = before;
        }
```

代码实现很简单，想当于把链表的当前元素来删掉。

既然上面在 put 的时候会根据最大容量来判断是否需要移除最不常用的元素了，下面我们就分析最常用的元素如何处理。内部原理就是当每次 get 的时候，如果找到了元素就把元素重新添加到链表的头部。

```java
public V get(Object key) {
        Entry<K,V> e = (Entry<K,V>)getEntry(key);
        if (e == null)
            return null;
        e.recordAccess(this);
        return e.value;
    }
```

代码超级简单， LinkedHashMap.Entry 重写了 recordAccess 方法，如下：

```java
void recordAccess(HashMap<K,V> m) {
            LinkedHashMap<K,V> lm = (LinkedHashMap<K,V>)m;
            if (lm.accessOrder) {
                lm.modCount++;
                remove();
                addBefore(lm.header);
            }
        }
```

可以看到先判断 accessOrder 这个成员变量，我们创建 LinkedHashMap 对象时候传入的是 true。里面的内部结构也很简单，先把自己移除，然后在把自己添加进去。

【参考资料】

1. jdk1.7


---EOF---
