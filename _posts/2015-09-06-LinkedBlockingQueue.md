---
layout: post
title: 浅析 LinkedBlockingQueue
categories: java基础
tags: java
---

在 Java 并发库当中提供了，LinkedBlockingQueue，使用链表来实现的阻塞队列。

简单介绍下实现：

- 内部使用单链表实现，有 Node 节点里面包含 next 元素和 item 存放当前数据
- 包含两个锁 takeLock 和 putLock，以及两个锁创建的条件 notEmpty 和 notFull
- 当 put 时候 putLock 上锁，比较 count 是否到达 capacity，如果到达 notFull 等待，没到达添加到链当中
- 当 take 时候 takeLock 上锁，比较 count 是否为 0，如果是 0，notEmpty 等待，没到达将元素从链当中去掉

## 源码分析

### 构造方法

```java
    private final int capacity;//  queue 容量

    private final AtomicInteger count = new AtomicInteger(0);// 存放元素个数

    private transient Node<E> head;// 链表头，item 永远是 null

    private transient Node<E> last;// 链表尾，next 永远是 null

    private final ReentrantLock takeLock = new ReentrantLock();

    private final Condition notEmpty = takeLock.newCondition();

    private final ReentrantLock putLock = new ReentrantLock();

    private final Condition notFull = putLock.newCondition();

static class Node<E> {
        E item;

        Node<E> next;

        Node(E x) { item = x; }
    }

public LinkedBlockingQueue() {
        this(Integer.MAX_VALUE);
    }

    public LinkedBlockingQueue(int capacity) {
        if (capacity <= 0) throw new IllegalArgumentException();
        this.capacity = capacity;
        last = head = new Node<E>(null);
    }

    public LinkedBlockingQueue(Collection<? extends E> c) {
        this(Integer.MAX_VALUE);
        final ReentrantLock putLock = this.putLock;
        putLock.lock(); // Never contended, but necessary for visibility
        try {
            int n = 0;
            for (E e : c) {
                if (e == null)
                    throw new NullPointerException();
                if (n == capacity)
                    throw new IllegalStateException("Queue full");
                enqueue(new Node<E>(e));
                ++n;
            }
            count.set(n);
        } finally {
            putLock.unlock();
        }
    }
```

上面列出了基本的属性和构造方法，比较简单主要看下第二个构造方法，初始化 capacity，然后把 last 和 head 都初始化到一个初始 node 上。

### put 方法

```java
public void put(E e) throws InterruptedException {
        if (e == null) throw new NullPointerException();
        int c = -1;
        Node<E> node = new Node(e);
        final ReentrantLock putLock = this.putLock;
        final AtomicInteger count = this.count;
        putLock.lockInterruptibly();
        try {
            while (count.get() == capacity) {
                notFull.await();
            }
            enqueue(node);
            c = count.getAndIncrement();
            if (c + 1 < capacity)
                notFull.signal();
        } finally {
            putLock.unlock();
        }
        if (c == 0)
            signalNotEmpty();
    }

private void enqueue(Node<E> node) {
        last = last.next = node;
    }
```

简单解释下 put 方法，先创建 node 元素，然后 putLock 尝试获取锁，如果获取到了循环查看当前 count 是不是到达了最大容量，如果到达了最大容量就 notFull 等待。如果没有到达那么执行 enqueue 方法添加 node 元素到链表。

enqueue 就一行代码，很精髓，如果是刚初始化完成的列表现在 head 和 last 都指向一个相同的 node，那么执行完这个代码后 head 的 next 就会是真实的第一个 node，然后 last 就指向这个第一个 node。如果是后续添加到队列，那么 last 就会向后移动，形成链表。

### take 方法

```java
public E take() throws InterruptedException {
        E x;
        int c = -1;
        final AtomicInteger count = this.count;
        final ReentrantLock takeLock = this.takeLock;
        takeLock.lockInterruptibly();
        try {
            while (count.get() == 0) {
                notEmpty.await();
            }
            x = dequeue();
            c = count.getAndDecrement();
            if (c > 1)
                notEmpty.signal();
        } finally {
            takeLock.unlock();
        }
        if (c == capacity)
            signalNotFull();
        return x;
    }

private E dequeue() {
        Node<E> h = head;
        Node<E> first = h.next;
        h.next = h; // help GC
        head = first;
        E x = first.item;
        first.item = null;
        return x;
    }
```

基本流程跟 put 一样，主要看下 dequeue 如何将元素移除到链表。首先会把 head 赋值到临时变量 h，h 的 next 其实就是我们要移除的元素，临时赋值给 first，并且把 h.next 指向 h，形成了一个环引用帮助 GC。然后把 first 赋值给 head，因为要返回元素，所以是 first 的 item 属性被返回，并且 head 的 item 永远是 null。

## 设计分析

1. 为什么要使用两个锁，takeLock 和 putLock ？
	- 因为在 put 和 take 我们操作的不同的链表对象，也就是操作的不同的 head 和 last，那么使用两个锁就是可以读和写并行。
2. signalNotEmpty 和  signalNotFull 方法的必要性
	- 在 put 方法当中如果队列加入了第一个元素就会去调用 signalNotEmpty 通知当前队列不是空的，避免了等待时间过长，在 take 方法当中的 signalNotFull 去通知当前队列不是满的一样。


---EOF---

