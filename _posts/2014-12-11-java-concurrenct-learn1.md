---
layout: post
title: Java并发学习1
categories: java基础
tags: 笔记
---

#### 线程和进程

在并发编程中，有两个基本概念：线程和进程。在 java 当中，并发主要关注线程，当然进程也很重要。

计算机系统有很多活跃的线程和进程。事实上系统只有一个执行核心，从而在同一时刻只有一个线程执行。处理核心的时间是通过一个操作系统进程和线程之间共享的特性称为时间切片。

计算机系统具有多个处理器或处理器的多个执行核变得越来越普遍。这大大增强了系统的进程和线程的并发执行 - 但即使在简单的系统并发是可能的，无需多个处理器或执行内核。

#### 进程

进程具有一个自包含的执行环境。一个进程通常有一个完整的，私有的一套基本的运行时的资源。特别是，每个进程具有其自身的存储空间。

流程往往被视为等同于程序或应用程序。然而，用户看到的内容作为一个单一应用可能实际上是一组协作的进程。为了便于进程之间的通信，大多数操作系统支持进程间通信（IPC）的资源，如 pipes 和 sockets。IPC 不只用在同一系统上的进程之间的通信，也可以在不同的系统。

大多数 java 虚拟机的实现值运行一个单一进程。Java 应用程序可以使用的 ProcessBuilder 对象创建额外的进程。

#### 线程

线程有时称为轻量级进程。进程和线程提供了一个执行环境，但创建一个新的线程，需要更少的资源比创建一个新的进程。

线程在进程当中 - 每个进程最少有一个线程。线程共享进程的资源，包括内存和打开的文件。这是高效的，但可能会引起问题在进行通信时。

多线程执行是 Java 平台的一个重要特征。每个应用程序都至少有一个线程 - 或几个，如果算上系统线程做的事情像内存管理和信号处理。但是从应用程序员的角度，你开始只有一个线程，称为主线程。

#### Thread 类

每个线程都是与 Thread 类的一个实例相关联。有两个方式使用 Thread 对象创建并发应用。

- 直接控制线程的创建和管理，每次当应用程序需要启动一个异步任务是时候创建一个线程对象。
- 从应用程序的其他部分抽象线程管理，通过 executor 来执行应用任务。

#### 定义和启动一个线程

创建线程有两种方法：

- 实现 Runnable 接口

```java
public class HelloRunnable implements Runnable {

    public void run() {
        System.out.println("Hello from a thread!");
    }

    public static void main(String args[]) {
        (new Thread(new HelloRunnable())).start();
    }

}
```

- 继承 Thread 类

```java
public class HelloThread extends Thread {

    public void run() {
        System.out.println("Hello from a thread!");
    }

    public static void main(String args[]) {
        (new HelloThread()).start();
    }

}
```

这两个例子都是调用 Thread.start 方法来启动一个新的线程。

如何使用这两种方式呢。第一种实现 Runnable 接口，更加通用，因为 Runnable 对象还可以继承其他类。第二种方式更加简单，但是会有只能是 Thread 类子类的限制。

Thread 类定义了一些方法，线程管理非常有用。

#### 暂停与睡眠线程

Thread.sleep 引起当前线程暂停执行指定的时间。这是使得处理器时间提供给其他应用程序或其他应用程序的问题可能会在计算机系统上运行的其他线程的有效手段。sleep 方法也可以用于心跳检查。

sleep 方法有两个重载的：一个是参数只传递睡眠的毫秒数，另外一个是传递睡眠指定的毫秒数加指定的纳秒数。然而，这些睡眠时间不能保证准确，此操作受到系统计时器和调度程序精度和准确性的影响。此外，睡眠周期可以通过中断终止。在任何情况下，你不能假设调用睡眠暂停的线程精确指定的时间段。

```java
public class SleepMessages {
    public static void main(String args[]) throws InterruptedException {
        String[] importantInfo = {
            "Mares eat oats",
            "Does eat oats",
            "Little lambs eat ivy",
            "A kid will eat ivy too"
        };

        for (int i = 0; i < importantInfo.length; i++) {
            //Pause for 4 seconds
            Thread.sleep(4000);
            //Print a message
            System.out.println(importantInfo[i]);
        }
    }
}
```

#### Interrupts

线程中断是指定当前线程停止现在正在做的事情并且去做其他事情。它是由程序员来决定究竟如何线程响应中断，但它是非常常见的线程终止。

调用 Thread.interrupt 方法来中断线程。为了中断机制能正常工作，被中断的线程必须支持自己的中断。

线程如何支持自己的中断呢。主要依赖于线程当前在做什么操作。如果线程频繁的调用会抛出 InterruptedException 异常的方法，他简单的 catch 住异常并且 return 就可以。

如下代码：

```java
for (int i = 0; i < importantInfo.length; i++) {
    // Pause for 4 seconds
    try {
        Thread.sleep(4000);
    } catch (InterruptedException e) {
        // We've been interrupted: no more messages.
        return;
    }
    // Print a message
    System.out.println(importantInfo[i]);
}
```

如果一个线程会执行很长时间，并且不抛出InterruptedException 异常。那么我们必须定期调用 Thread.interrupted 方法，如果当前线程已经中断，则返回 true；否则返回 false。

```java
for (int i = 0; i < inputs.length; i++) {
    heavyCrunch(inputs[i]);
    if (Thread.interrupted()) {
        // We've been interrupted: no more crunching.
        return;
    }
}
```

在这个简单的例子中，代码简单地测试该线程是否中断。在更复杂的应用程序，它可能会更有意义抛出一个 InterruptedException：

```java
if (Thread.interrupted()) {
    throw new InterruptedException();
}
```

这使得中断处理代码在 catch 子句集中。

#### 中断状态标志

中断的机制是使用一个中断的状态来判断现在是否中断。调用 Thread.interrupt 改变这个状态。当通过调用静态方法 Thread.interrupted 线程检查中断，中断状态被清除。非静态的方法 isInterrupted，这是由一个线程来查询的另一个中断状态，不改变中断状态标志。

#### Joins

join 方法允许一个线程等待另一个线程完成。调用 t.join(); 导致当前线程暂停执行，直到T的线程终止。重载的 join 方法可以指定等待的时间。sleep，join 方法都会抛出 InterruptedException 异常。

#### 简单的线程例子

```java
public class SimpleThreads {

    // 打印当前线程名字和message
    static void threadMessage(String message) {
        String threadName = Thread.currentThread().getName();
        System.out.format("%s: %s%n", threadName, message);
    }

    private static class MessageLoop implements Runnable {
        public void run() {
            String importantInfo[] = {
                "Mares eat oats",
                "Does eat oats",
                "Little lambs eat ivy",
                "A kid will eat ivy too"
            };
            try {
                for (int i = 0; i < importantInfo.length; i++) {
                    // Pause for 4 seconds
                    Thread.sleep(4000);
                    // Print a message
                    threadMessage(importantInfo[i]);
                }
            } catch (InterruptedException e) {
                threadMessage("I wasn't done!");
            }
        }
    }

    public static void main(String args[]) throws InterruptedException {

        long patience = 10000;

        threadMessage("Starting MessageLoop thread");
        long startTime = System.currentTimeMillis();
        Thread t = new Thread(new MessageLoop());
        t.start();

        threadMessage("Waiting for MessageLoop thread to finish");

        while (t.isAlive()) {
            threadMessage("Still waiting...");
            t.join(1000);
            if (((System.currentTimeMillis() - startTime) > patience)
                  && t.isAlive()) {
                threadMessage("Tired of waiting!");
                t.interrupt();
                // Shouldn't be long now
                // -- wait indefinitely
                t.join();
            }
        }
        threadMessage("Finally!");
    }
}
```

#### 同步

线程之间的通信主要是访问共享变量。这个非常有效的，但使两种可能的错误：线程干扰和内存一致性错误。防止这些错误的工具是同步。

#### 线程干扰

```java
class Counter {
    private int c = 0;

    public void increment() {
        c++;
    }

    public void decrement() {
        c--;
    }

    public int value() {
        return c;
    }

}
```

Counter 类是用来计数的，但是当多线程共同访问同一个 Counter 对象的时候会出现问题。因为 c++ 和 c-- 不是原子操作。一般会分成三步：

- 读取当前 c 的值
- 给 c 增加1或者减1
- 将新值重新存到 c 中

#### 内存一致性问题

不同线程访问相同数据但是看到的结果不一样会引起内存一致性错误。

要避免内存一致性错误需要理解 happens-before 关系。这种关系仅仅是保证内存写入一个特定的语句中可以看到另一特定声明 。看下面的例子：

```java
int counter = 0;
counter++;
System.out.println(counter);
```

如果线程 A 和线程 B 可以共享 counter 变量。如果上面的代码块在同一个线程当中执行是线程安全的，会打印出1。但是如果在两个线程中同时执行就可能都打印出1。是因为没有保证 A 线程的改变对 B 线程可见。除非建立了 happens-before 关系在这两个代码块之间。

有几种创建 happens-before 关系的方式，其中一个是 synchronized 。

我们已经看到了两种方式创建 happens-before 关系。

- 当调用 Thread.start 方法的时候，会创建 happens-before 关系，则会导致原来的代码对新线程是可见的。
- 当一个线程终止，导致 Thread.join 的另一个线程返回，那么所有的终止线程执行的语句与所有成功 join 的语句有 happens-before 关系，之前的关系。效果是在执行 join 的时候是可见的。

#### Synchronized  方法

java 提供了两种同步的方式：synchronized 方法和 synchronized 代码块。

看下面的例子：

```java
public class SynchronizedCounter {
    private int c = 0;

    public synchronized void increment() {
        c++;
    }

    public synchronized void decrement() {
        c--;
    }

    public synchronized int value() {
        return c;
    }
}
```

在方法上加上了 synchronized 关键字有两个作用：

- 在同一个对象上不可能同时调用两个同步方法。当一个线程执行了一个对象的同步方法，其他的线程想要执行这个对象的同步方法时候会被阻塞，直到第一个线程执行完成。
- 当同步方法存在，他自动建立 happens-before 关系为后续的 synchronized 方法调用。这保证了改变对象的状态对所有线程是可见的。

注意：构造方法不能使用 synchronized 关键字。

synchronized 方法是防止线程干扰和内存一致性错误的简单策略：如果一个对象可以被多个线程访问，那么所有的读或者写操作都是通过 synchronized 方法。

#### 内在的锁和同步

同步构建在内在锁或监视器锁基础之上的。内在锁起到同步两方面的作用：强制独占访问对象的状态和建立 happens-before 关系保证可见性。

每一个对象都有一个与其相关联的内置锁。默认情况下，一个线程需要强制独占对象的属性必须在访问之间获取这个对象的内置锁，执行完之后释放这个锁。只要一个线程拥有了内置锁，其他的线程都不能获取相同的锁。

当一个线程释放了内置锁，一个 happens-before 关系就会被创建在这个操作和后续相同锁。

#### 在 synchronized 方法中的锁

当一个线程执行 synchronized 方法吗，他自动获取的内部锁为这个方法的对象并且当方法返回被释放。当有未捕获的异常导致的返回也会释放锁。

你可能调用一个静态的 synchronized 方法，静态的方法会锁住这个对象的 class 对象。

#### synchronized 代码块

如下代码：

```java
public void addName(String name) {
    synchronized(this) {
        lastName = name;
        nameCount++;
    }
    nameList.add(name);
}
```

synchronized 代码块可以有效的改善同步的力度。有一个类 MsLunch，里面有两个属性，c1 和 c2，并且这两个属性不会一起使用。所有对属性的更新操作都必须是同步的，但是没有理由阻止更新 c1 后才可以更新 c2，这样会没必要的造成并发阻塞。代替 synchronized 方法和 this 对象，我们创建两个对象来充当锁。

```java
public class MsLunch {
    private long c1 = 0;
    private long c2 = 0;
    private Object lock1 = new Object();
    private Object lock2 = new Object();

    public void inc1() {
        synchronized(lock1) {
            c1++;
        }
    }

    public void inc2() {
        synchronized(lock2) {
            c2++;
        }
    }
}
```

#### Atomic 访问

在程序中，原子操作是要么成功要么失败的，不会在中间停止。

我们已经看到了自增操作不是原子操作。现在，可以指定这些操作是原子操作：

- 写和读是原子操作为引用类型变量和大多数基本类型变量。
- 写和读是原子操作为所有变量声明成 volatile。

原子操作不能被插入，所以不会出现线程干扰。然而，这并没有消除所有需要同步的原子操作，因为内存一致性错误仍然是可能的。使用 volatile 变量降低了内存一致性错误的风险，因为任何写入 volatile 变量会建立 happens-before 关系为后续读取相同的变量。这意味着修改 volatile 变量对其他线程总是可见的。更重要的是线程读取的  volatile 变量，它看到的不仅仅是这最后变化的，也会引起改变代码的副作用。

使用简单的原子变量访问比同步代码访问更有效，但是需要注意内存一致性问题。

【参考资料】

- [oracle并发教程](http://docs.oracle.com/javase/tutorial/essential/concurrency/memconsist.html)

---EOF---

