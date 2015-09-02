---
layout: post
title: 浅析 ArrayBlockingQueue
categories: java基础
tags: java
---

在 Java 并发库当中提供了，ArrayBlockingQueue，使用数组来实现的阻塞队列。

简单介绍下实现：

- 内部使用数组存储元素
- 使用 ReentrantLock 来保证线程安全
- 使用两个 Condition 来进行 notEmpty 和 notFull 的阻塞操作
- 使用 takeIndex 和 putIndex 来指定获取元素和存元素的下标

## 源码分析

### 构造方法

```java
public ArrayBlockingQueue(int capacity) {
        this(capacity, false);
    }

public ArrayBlockingQueue(int capacity, boolean fair) {
        if (capacity <= 0)
            throw new IllegalArgumentException();
        this.items = new Object[capacity];
        lock = new ReentrantLock(fair);
        notEmpty = lock.newCondition();
        notFull =  lock.newCondition();
    }

public ArrayBlockingQueue(int capacity, boolean fair,
                              Collection<? extends E> c) {
        this(capacity, fair);

        final ReentrantLock lock = this.lock;
        lock.lock(); // Lock only for visibility, not mutual exclusion
        try {
            int i = 0;
            try {
                for (E e : c) {
                    checkNotNull(e);
                    items[i++] = e;
                }
            } catch (ArrayIndexOutOfBoundsException ex) {
                throw new IllegalArgumentException();
            }
            count = i;
            putIndex = (i == capacity) ? 0 : i;
        } finally {
            lock.unlock();
        }
    }
```

看构造方法比较简单，主要是在构造的时候创建了一个 ReentrantLock 和两个 Condition。

### put 方法

```java
public void put(E e) throws InterruptedException {
        checkNotNull(e);
        final ReentrantLock lock = this.lock;
        lock.lockInterruptibly();
        try {
            while (count == items.length)
                notFull.await();
            insert(e);
        } finally {
            lock.unlock();
        }
    }

private void insert(E x) {
        items[putIndex] = x;
        putIndex = inc(putIndex);
        ++count;
        notEmpty.signal();
    }

final int inc(int i) {
        return (++i == items.length) ? 0 : i;
}
```

首先我们在 put 一个元素的时候会使用 `lock.lockInterruptibly()`，当前线程没有被中断的时候获取锁。

然后判断 count (当前存的元素个数) 是否与数组长度相等，如果相等 notFull 进行等待。这样我们在 put 的时候如果队列满了就会进行阻塞，使用 while 循环是因为 notFull 可能被假唤醒，避免了被假唤醒后出现数组越界。

最后看下 insert 方法就是把元素放在数组内，下标就是 putIndex，存放完会更新 putIndex，并且会 notEmpty 信号释放。putIndex 的增加规则就是如果值等于了数组的长度那么就从 0 重新开始。

### take 方法

```java
public E take() throws InterruptedException {
        final ReentrantLock lock = this.lock;
        lock.lockInterruptibly();
        try {
            while (count == 0)
                notEmpty.await();
            return extract();
        } finally {
            lock.unlock();
        }
    }

private E extract() {
        final Object[] items = this.items;
        E x = this.<E>cast(items[takeIndex]);
        items[takeIndex] = null;
        takeIndex = inc(takeIndex);
        --count;
        notFull.signal();
        return x;
    }

final int inc(int i) {
        return (++i == items.length) ? 0 : i;
    }
```

take 的步骤基本跟 put 相同，就不详细描述。


---EOF---

