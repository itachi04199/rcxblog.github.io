---
layout: post
title: Jvm 垃圾收集器 CMS & G1
categories: JVM
tags: 笔记 java
---

## CMS(Concurrent Mark Sweep) 收集器

CMS 收集器是获取最短回收停顿时间为目标的收集器。适用于 B/S 系统的服务端。从名字上可以看出来是使用的标记清除算法。整个清除过程如下：

- 初始标记（CMS initial mark）
- 并发标记（CMS concurrent mark）
- 重新标记（CMS remark）
- 并发清除（CMS concurrent sweep）

其中初始标记和重新标记依然会停止所有的用户线程。

初始标记仅仅是标记一下 GC Roots 能直接关联到的对象，速度很快。

并发标记阶段就是进行 GC Roots Tracing 的过程。

重新标记是为了修正并发标记期间，因用户程序继续运行而导致标记产生变动的那一部分对象的标记记录，这个阶段的停顿时间一般比初始标记阶段长。

整个 GC 过程当中消耗时间最长的是并发标记和并发清除过程，但是这两个阶段的垃圾回收线程可以与用户线程一起并发执行，总体上说，CMS 收集器的内存回收过程是与用户线程一起并发执行的。

![](http://renchx.com/public/images/jvm-cms.png)

**CMS缺点:**

- CMS 对 CPU 资源敏感，在并发阶段虽然不会停止用户线程，但是会占用一部分线程来进行垃圾回收，总吞吐量会降低。CMS 默认启动的线程是（CPU数量+3)/4，当 CPU 大于4个以上占用资源不超过 25% 的 CPU 资源，但是小于 4 个 CPU 时候 CMS 收集器对用户程序的影响就比较大。
- CMS 无法回收浮动垃圾。CMS 在并发清理阶段还可以运行用户线程，这时候还会产生新的垃圾，而这部分垃圾 CMS 无法在本次回收掉，这部分就是浮动垃圾。因此 CMS 不能像其他的收集器等到年老代几乎全部满了再进行回收，需要预留一部分空间提供并发收集时候的用户线程使用。默认设置下，CMS 收集器在年老代使用了 68% 的空间后就会被激活，可以通过 -XX:CMSInitiatingOccupancyFraction 参数来设置这个属性。如果 CMS 在运行时候预留的内存无法满足程序需要，就会出现一次“Concurrent Mode Failure”失败，这时候虚拟机临时启用 Serial Old 收集器重新来进行老年代的垃圾收集。
- CMS 是基于标记整理算法，在清理的过程中会有大量的空间碎片。空间碎片过多后给打对象分配空间会有很多麻烦。CMS 提供了一个参数 -XX:+UseCMSCompactAtFullCollection 用来在 Full GC 完成后附加一个碎片整理过程，碎片整理无法并发会导致停顿时间变长。当然还提供了一个参数 -XX:CMSFullGCsBeforeCompaction，这个参数设置在执行多少次不压缩的 Full GC 后，跟着来一次带压缩的。

## G1 收集器

G1 是一款面向服务端应用的垃圾收集器。有如下特点：

- 并行与并发：G1 可以充分利用多 CPU 、多核环境下的硬件优势，使用多 CPU 来缩短用户线程的停顿时间。
- 分代收集：G1 可以不需要其他收集器配合就可以独立管理整个 GC 堆，但是依旧保留了分代概念。
- 空间整合：G1 从整体上看是基于标记整理算法，从局部（两个 Region 之间）看是基于复制算法实现，这两种算法都不会产生内存碎片。
- 可预测的停顿：G1	可以让使用者明确指定一个长度为 M 毫秒的时间片段内，消耗在垃圾回收的时间不超过 N 毫秒。

使用 G1 收集器，Java 堆的内存布局与其他的收集器有很大区别，它将整个 Java 堆划分成多个大小相等的独立区域（Region），虽然还是有年轻代和老年代的概念，但是新生代和老年代不在是物理隔离的，它们都是一部分 Region(不需要连续) 的集合。

G1 尽量避免在 Java 堆中进行全区域的垃圾回收。G1 跟踪各个 Region 里面的垃圾堆积的价值大小（回收说获得的空间大小以及回收所需要时间的经验值），在后台维护一个优先列表，每次根据允许的收集时间，优先回收价值最大的 Region（这也是 Gabage-First 名字的由来）。这样可以提高回收垃圾的效率。

G1 收集器运作大致有几个步骤：

- 初始标记
- 并发标记
- 最终标记
- 筛选回收

【参考资料】

1. [深入理解Java虚拟机](http://book.douban.com/subject/24722612/)

---EOF---
