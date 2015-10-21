---
layout: post
title: Java 对象的创建与布局
categories: JVM
tags: 笔记 java
---

## 对象的创建

虚拟机遇到 new 指令，首先检查这个指令的参数是否能在常量池中定位到一个类的符号引用，并且检查这个符号引用是否已经被加载、解析、初始化过。

在类加载完成后给对象分配内存。分配内存一般有两种方式：指针碰撞和空闲列表。指针碰撞是只 Java 堆当中的内存是规则的，已使用内存在一边，未使用内存在另外一边，中间放着一个指针作为分界点的指示器，如果需要分配内存就只需要把指针向空闲空间那边挪动一段与对象大小相等的距离。空闲列表是只已用内存和空闲内存是交互存在的，这样虚拟机就必须维护一个列表，记录哪块内存是可用的，在分配内存的时候从列表中找到一个足够大的空间划分给对象实例，并且更新空闲列表。

一般 Serial、ParNew 收集器采用复制算法，所以系统采用的对象内存分配算法是指针碰撞，而 CMS 基于标记清除算法，通常采用空闲列表。

问题：并发创建对象会出现指针没来得及修改导致的重复地址覆盖对象。

解决方案：

- 分配内存动作同步-采用 CAS 方式保证更新操作的原子性
- 预先给 Java 线程在堆中分配一小块内存，称为本地线程分配缓冲（TLAB），只有当该线程用完才需要同步锁。是否使用 TLAB 可以通过 -XX:+/-UseTLAB

内存分配完成后，虚拟机需要把分配到的内存空间都初始化为零值（不包括对象头）。

接下来，虚拟机要对对象进行必要的设置，例如这个对象是哪个类的实例、如何才能早到类的元数据信息、对象的哈希码、对象的 GC 分代年龄等信息。这些信息存放在对象的对象头当中。

上面工作完成后，从虚拟机来看，一个新的对象已经产生了，但从 Java 程序的视角来看，对象创建才刚刚开始--`<init>`方法还没执行，所有字段还都是零。一般来说（由字节码是否跟随 invokespecial 指令决定），执行 new 指令后会接着执行 init 方法。

```java
public class TT {
	public static void main(String[] args) {
		Object object = new Object();
	}
}

public static main([Ljava/lang/String;)V
   L0
    LINENUMBER 4 L0
    NEW java/lang/Object // NEW 指令
    DUP
    INVOKESPECIAL java/lang/Object.<init> ()V //跟随了 INVOKESPECIAL 指令
    ASTORE 1
   L1
    LINENUMBER 5 L1
    RETURN
   L2
    LOCALVARIABLE args [Ljava/lang/String; L0 L2 0
    LOCALVARIABLE object Ljava/lang/Object; L1 L2 1
    MAXSTACK = 2
    MAXLOCALS = 2
```

## 对象的内存布局

在 HotSop 虚拟机中，对象在内存中存储的布局可以分为 3 块区域：对象头、实例数据、对齐填充。

对象头包括两部分，一部分是用于存储对象自身的运行时数据，如对象的哈希码、对象的 GC 分代年龄、锁状态标志、线程持有的锁、偏向线程 ID 等。另一部分是类型指针，即对象指向它的类型元数据的指针，虚拟机通过这个指针来确定这个对象是哪个类的实例。并不是所有虚拟机实现都必须在对象上保留类型指针，如果对象是数组，那在对象头中还必须有一块用于记录数组长度的数据，因为虚拟机可以通过普通 Java 对象的元数据确定 Java 对象的大小，但是从数组的元数据无法确定数组的大小。

实例数据是对象真正存储的有效信息，也是程序代码中所定义的各种类型的字段内容。

对齐填充不一定存在，也没特殊含义，仅仅是占位符作用，由于 HotSpot VM 管理对象需要对象是 8 字节的整数倍，所以通过对齐填充补齐。

## 对象的访问定位

两种方式：使用句柄、直接指针。

![](http://renchx.com/public/images/jvm-ref1.png)

![](http://renchx.com/public/images/jvm-ref2.png)


【参考资料】

1. [深入理解Java虚拟机](http://book.douban.com/subject/24722612/)

---EOF---
