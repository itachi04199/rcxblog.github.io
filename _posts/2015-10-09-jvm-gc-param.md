---
layout: post
title: Jvm gc 常用参数记录
categories: JVM
tags: 笔记 java
---

## 关于JVM选项的几点：

- 布尔型参数选项：-XX:+ 打开， -XX:- 关闭。（比如-XX:+PrintGCDetails）
- 数字型参数选项通过-XX:=设定。数字可以是 m/M(兆字节)，k/K(千字节)，g/G(G字节)。比如：32K表示32768字节。（-XX:NewSize=256m）
- 字符行参数选项通过-XX:=设定，通常用来指定一个文件，路径，或者一个命令列表。（-XX:HeapDumpPath=./java_pid.hprof）
命令 java -help可以列出java 应用启动时标准选项（不同的JVM实现是不同的）。java -X可以列出不标准的参数（这是JVM的扩展特性）。-X相关的选项不是标准的，被改变也不会通知。如果你想查看当前应用使用的JVM参数，你可以使用：ManagementFactory.getRuntimeMXBean().getInputArguments()。


## 堆内存设置（大小设置）

- -Xms：初始堆大小
- -Xmx：最大堆大小
- -Xmn：新生代的内存空间大小，也可以使用 -XX:NewSize 设置。
- -XX:SurvivorRatio：新生代中 Eden 和 Survivor 的容量比例，默认是8。
- -Xss：每个线程的堆栈大小。
- -XX:PermSize：永久代初始值。
- -XX:MaxPermSize：永久代最大值。
- -XX:NewRatio：年轻代(包括Eden和两个Survivor区)与年老代的比值(除去持久代)

## Gc日志参数

通过在 tomcat 启动脚本中添加相关参数生成 gc 日志。

- -verbose:gc 开关可显示 GC 的操作内容。打开它，可以显示最忙和最空闲收集行为发生的时间、收集前后的内存大小、收集需要的时间等。
- 打开 -XX:+PrintGCdetails 开关，可以详细了解 GC 中的变化。
- 打开 -XX:+PrintGCTimeStamps 开关，可以了解这些垃圾收集发生的时间，自 JVM 启动以后以秒计量。
- -Xloggc:$CATALINA_BASE/logs/gc.log gc日志产生的路径
- -XX:+PrintGCApplicationStoppedTime 输出GC造成应用暂停的时间
- -XX:+PrintGCDateStamps GC发生的时间信息

## Gc日志分析工具

### GC日志格式

```
2012-11-15T16:57:12.524+0800: 8.490 : [GC 8.490: [ParNew: 118016K->11244K(118016K), 0.0525525 secs] 183413K->83007K(511232K), 0.0527229 secs] [Times: user=0.08 sys=0.00, real=0.05 secs]
```

1. 8.490：表示虚拟机启动运行到8.490秒是进行了一次monor Gc(not Full GC)
2. ParNew: 表示对年轻代进行的GC，使用ParNew收集器
3. 118016K->11244K(118016K)：118016K 年轻代收集前大小，11244K 收集完以后的大小，118016K 当前年轻代分配的总大小
4. 0.0525525 secs：表示对年轻代进行垃圾收集时，用户线程暂停的时间，即此次年轻代收集花费的时间
5. 183413K->83007K(511232K):JVM heap堆收集前后heap堆内存的变化
6. 0.0527229 secs：整个JVM此次垃圾造成用户线程的暂停时间。

更全一点的参数说明：

```
[GC [<collector>: <starting occupancy1> -> <ending occupancy1>, <pause time1> secs] <starting occupancy3> -> <ending occupancy3>, <pause time3> secs]

<collector>              GC收集器的名称
<starting occupancy1>    新生代在GC前占用的内存
<ending occupancy1>      新生代在GC后占用的内存
<pause time1>            新生代局部收集时jvm暂停处理的时间
<starting occupancy3>    JVM Heap 在GC前占用的内存
<ending occupancy3>      JVM Heap 在GC后占用的内存
<pause time3>            GC过程中jvm暂停处理的总时间
```

【参考资料】

1. http://xstarcd.github.io/wiki/Java/JVM_GC.html
2. http://www.oschina.net/translate/hotspot-jvm-options-java-examples
