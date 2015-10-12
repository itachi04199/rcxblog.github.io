---
layout: post
title: Jvm 垃圾收集器1
categories: JVM
tags: 笔记 java
---

JVM 的垃圾回收器有很多种，先看下面的图例:

![jvm-types.png](http://renchx.com/public/images/jvm-types.png)

## Serial(串行) 收集器

从名字可以看出来这是一个单线程的垃圾收集器，这个单线程不仅仅是使用一个 CPU 或一条收集线程去完成，而且在垃圾收集的时候必须暂停所有其他的工作线程。

![jvm-serial.png](http://renchx.com/public/images/jvm-serial.png)

主要用在 Client 模式下的默认新生代收集器，优势是简单而高效。

## ParNew 收集器

ParNew 其实就是 Serial 的多线程版本，除了使用多线程进行垃圾搜集外，其余的控制参数、收集算法、对象分配规则、回收策略都与 Serial 一样。

ParNew 是许多运行在 Server 模式下的虚拟机首选的新生代收集器。因为只有 Serial 和 ParNew 可以与 CMS 组合使用。

ParNew 也是使用 -XX:+UseConcMarkSweepGC 选项后的默认新生代收集器，也可以使用 -XX:+UseParNewGC 来指定。

![jvm-parnew.png](http://renchx.com/public/images/jvm-parnew.png)

**Tips**

- 并行(Parallel)：指多条垃圾收集线程并行工作，但是用户线程还是处于等待状态。
- 并发(Concurrent)：指用户线程与垃圾收集线程同时执行。

## Parallel Scavenge 收集器

 Parallel Scavenge 是一个新生代收集器，也是使用复制算法，也是并行多线程。

 Parallel Scavenge 与 ParNew 的不同点是， Parallel Scavenge 收集器的目标是达到一个可控制的吞吐量，而其他的收集器关注的是尽可能缩短垃圾收集时候用户线程的停顿时间。吞吐量是 CPU 用于运行用户代码的时间与 CPU 总消耗时间的比值，虚拟机运行了 100 分钟，垃圾回收消耗了 1 分钟，那么吞吐率就是 99%。

停顿时间短适合需要跟用户交互的程序，高吞吐量则可以高效利用 CPU 时间，主要适合后台运算不需要太多交互的任务。

 Parallel Scavenge 收集器提供了两个参数用于精确控制吞吐量，分别是控制最大垃圾收集啊停顿时间的 -XX:MaxGCPauseMilis 参数以及直接设置吞吐量大小的 -XX:GCTimeRatio 参数。

MaxGCPauseMilis 参数允许的值是一个大于 0 的毫秒数，收集器尽量控制回收垃圾时间不超过设定值。但是这个值不要设置太小，因为 GC 停顿时间缩短是牺牲吞吐量和新生代空间来换取的。系统把新生代调小，收集 300M 肯定比 500M 快，也直接导致了垃圾收集更加频繁，原来10秒一次、每次停顿100ms，现在5秒收集一次，每次停顿70ms。停顿时间下降了，但是吞吐量也下降了。

GCTimeRatio 参数是 0-100 的整数，也就是垃圾收集时间占中时间的比率的底数。如果参数设置成 19，那允许的最大 GC 时间就占 1/(1+19) 即 5%。默认是 99 ，允许 1 / (1+99) 的 GC 时间。

-XX:+UseAdaptiveSizePolicy 参数打开后，不需要手动指定 Eden 与 Survivor 的比例、晋升老年代对象年龄等细节，虚拟机会自动调节大小。

**Tips**

不能与 CMS 组合使用。

## Serial Old 收集器

Serial 收集器的老年代版，一样是单线程，使用标记整理算法。

## Parallel Old 收集器

Parallel Old 是 Parallel Scavenge 的老年代版本，使用多线程了标记整理算法。

在 JDK 1.6 前，Parallel Scavenge 收集器的位置很尴尬，因为他不能与 CMS 组合，所以当新生代选择了 Parallel Scavenge ，老年代必须选择 Serial Old 收集器。

【参考资料】

1. [深入理解Java虚拟机](http://book.douban.com/subject/24722612/)

---EOF---