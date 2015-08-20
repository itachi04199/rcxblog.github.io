---
layout: post
title: java 当中的 Queue 学习
categories: java基础
tags: 笔记, java
---

Queue 是队列的意思。队列是一种特殊的线性表，它只允许在表的前端（front）进行删除操作，而在表的后端（rear）进行插入操作。进行插入操作的端称为队尾，进行删除操作的端称为队头。队列中没有元素时，称为空队列。

## Queue 接口

下面是 JDK 当中 Queue 接口：

```java
public interface Queue<E> extends Collection<E> {

    boolean add(E e);

    boolean offer(E e);

    E remove();

    E poll();

    E element();

    E peek();
}
```

我们看 JDK 文档对这个 Queue 接口方法的描述：

每个方法都存在两种形式：一种抛出异常（操作失败时），另一种返回一个特殊值（null 或 false，具体取决于操作）。插入操作的后一种形式是用于专门为有容量限制的 Queue 实现设计的；在大多数实现中，插入操作不会失败。

操作|抛出异常|返回特殊值
:--|:--|:--
插入|add(e)|offer(e)
删除|remove()|poll()
检查|element()|peek()

简单说明下上面的表，例如 add(e) 的时候 Queue 队列满了，则会抛出异常。offer(e) 的时候 Queue 队列满了，会返回 false。

## BlockingQueue

下面的 BlockingQueue 的源码：

```java
public interface BlockingQueue<E> extends Queue<E> {
    boolean add(E e);

    boolean offer(E e);

    void put(E e) throws InterruptedException;

    boolean offer(E e, long timeout, TimeUnit unit)
        throws InterruptedException;

    E take() throws InterruptedException;

    E poll(long timeout, TimeUnit unit)
        throws InterruptedException;

    int remainingCapacity();

    boolean remove(Object o);

    public boolean contains(Object o);

    int drainTo(Collection<? super E> c);

    int drainTo(Collection<? super E> c, int maxElements);
}
```

阻塞队列是支持两个附加操作的 Queue。这两个操作是：获取元素时等待队列变为非空，以及存储元素时等待空间变得可用。

BlockingQueue 方法以四种形式出现，对于不能立即满足但可能在将来某一时刻可以满足的操作，这四种形式的处理方式不同：第一种是抛出一个异常，第二种是返回一个特殊值（null 或 false，具体取决于操作），第三种是在操作可以成功前，无限期地阻塞当前线程，第四种是在放弃前只在给定的最大时间限制内阻塞。下表中总结了这些方法：

操作|抛出异常|	特殊值|	阻塞|	超时
:--|:--|:--|:--
插入|	add(e)|	offer(e)|	put(e)|	offer(e, time, unit)
移除|	remove()|	poll()|	take()|	poll(time, unit)
检查|	element()|	peek()|	不可用|	不可用

BlockingQueue 不接受 null 元素。试图 add、put 或 offer 一个 null 元素时，某些实现会抛出 NullPointerException。null 被用作指示 poll 操作失败的警戒值。

---EOF---