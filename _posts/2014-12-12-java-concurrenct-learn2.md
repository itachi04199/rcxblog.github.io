---
layout: post
title: Java并发学习2
categories: java基础
tags: 笔记
---

#### 死锁

死锁描述的情况是两个或多个线程被阻塞永远等待对方。下面是一个例子。

```java
public class Deadlock {
    static class Friend {
        private final String name;
        public Friend(String name) {
            this.name = name;
        }
        public String getName() {
            return this.name;
        }
        public synchronized void bow(Friend bower) {
            System.out.format("%s: %s"
                + "  has bowed to me!%n",
                this.name, bower.getName());
            bower.bowBack(this);
        }
        public synchronized void bowBack(Friend bower) {
            System.out.format("%s: %s"
                + " has bowed back to me!%n",
                this.name, bower.getName());
        }
    }

    public static void main(String[] args) {
        final Friend alphonse =
            new Friend("Alphonse");
        final Friend gaston =
            new Friend("Gaston");
        new Thread(new Runnable() {
            public void run() { alphonse.bow(gaston); }
        }).start();
        new Thread(new Runnable() {
            public void run() { gaston.bow(alphonse); }
        }).start();
    }
}
```

在上面的例子当中这两个线程都在等待另外的线程退出后获取相应的锁，但是谁都在没获得锁之前不会退出，所以就死锁了。

#### 饥饿

饥饿描述了一种情况，一个线程不能访问共享资源来执行流程。这经常发生在资源被线程独占很长时间。例如，假设一个对象提高了一个 synchronized 方法并且运行很长时间才会返回。如果一个线程频繁的调用这个方法，其他线程也频繁的调用这个方法会导致经常被阻塞。

#### 活锁

一个线程通常会有会响应其他线程的活动。如果其他线程也会响应另一个线程的活动，那么就有可能发生活锁。同死锁一样，发生活锁的线程无法继续执行。然而线程并没有阻塞——他们在忙于响应对方无法恢复工作。这就相当于两个在走廊相遇的人：Alphonse 向他自己的左边靠想让 Gaston 过去，而 Gaston 向他的右边靠想让 Alphonse 过去。可见他们阻塞了对方。Alphonse 向他的右边靠，而 Gaston 向他的左边靠，他们还是阻塞了对方。

#### Guarded Blocks

线程之间经常进行协调工作，最常见的方式是使用 Guarded Blocks，它循环检查一个条件（通常初始值为 true），直到条件发生变化才跳出循环继续执行。在使用 Guarded Blocks 时有以下几个步骤需要注意：

假设 guardedJoy() 方法必须要等待另一线程为共享变量joy设值才能继续执行。那么理论上可以用一个简单的条件循环来实现，但在等待过程中 guardedJoy 方法不停的检查循环条件实际上是一种资源浪费。

```java
public void guardedJoy() {
    // Simple loop guard. Wastes
    // processor time. Don't do this!
    while(!joy) {}
    System.out.println("Joy has been achieved!");
}
```

更加高效的方法是调用 Object.wait 将当前线程挂起，直到有另一线程发起事件通知（尽管通知的事件不一定是当前线程等待的事件）。

```java
public synchronized void guardedJoy() {
    // This guard only loops once for each special event, which may not
    // be the event we're waiting for.
    while(!joy) {
        try {
            wait();
        } catch (InterruptedException e) {}
    }
    System.out.println("Joy and efficiency have been achieved!");
}
```

注意：一定要在循环里面调用 wait 方法，不要被唤醒后 joy 的值不一定被改变。

和其他可以暂停线程执行的方法一样，wait 方法会抛出 InterruptedException，在上面的例子中，因为我们关心的 是joy 的值，所以忽略了 InterruptedException。

为什么 guardedJoy 是 synchronized 方法？假设 d 是用来调用 wait 的对象，当一个线程调用 d.wait，它必须要拥有 d 的内部锁（否则会抛出异常），获得 d 的内部锁的最简单方法是在一个 synchronized 方法里面调用 wait。

当一个线程调用 wait 方法时，它释放锁并挂起。然后另一个线程请求并获得这个锁并调用 Object.notifyAll 通知所有等待该锁的线程。

```java
public synchronized notifyJoy() {
    joy = true;
    notifyAll();
}
```

当第二个线程释放这个该锁后，第一个线程再次请求该锁，从 wait 方法返回并继续执行。

注意：还有另外一个通知方法，notify()，它只会唤醒一个线程。但由于它并不允许指定哪一个线程被唤醒，所以一般只在大规模并发应用（即系统有大量相似任务的线程）中使用。因为对于大规模并发应用，我们其实并不关心哪一个线程被唤醒。

现在我们使用 Guarded blocks 创建一个生产者/消费者应用。这类应用需要在两个线程之间共享数据：生产者生产数据，消费者使用数据。两个线程通过共享对象通信。在这里，线程协同工作的关键是：生产者发布数据之前，消费者不能够去读取数据；消费者没有读取旧数据前，生产者不能发布新数据。

在下面的例子中，数据通过 Drop 对象共享的一系列文本消息：

```java
public class Drop {

    private String message;

    private boolean empty = true;

    public synchronized String take() {
        while (empty) {
            try {
                wait();
            } catch (InterruptedException e) {}
        }
        empty = true;
        notifyAll();
        return message;
    }

    public synchronized void put(String message) {
        while (!empty) {
            try {
                wait();
            } catch (InterruptedException e) {}
        }
        empty = false;
        this.message = message;
        notifyAll();
    }
}
```

Producer 是生产者线程，发送一组消息，字符串 DONE 表示所有消息都已经发送完成。为了模拟现实情况，生产者线程还会在消息发送时随机的暂停。

```java
import java.util.Random;

public class Producer implements Runnable {
    private Drop drop;

    public Producer(Drop drop) {
        this.drop = drop;
    }

    public void run() {
        String importantInfo[] = {
            "Mares eat oats",
            "Does eat oats",
            "Little lambs eat ivy",
            "A kid will eat ivy too"
        };
        Random random = new Random();

        for (int i = 0; i < importantInfo.length; i++) {
            drop.put(importantInfo[i]);
            try {
                Thread.sleep(random.nextInt(5000));
            } catch (InterruptedException e) {}
        }
        drop.put("DONE");
    }
}
```

Consumer 是消费者线程，读取消息并打印出来，直到读取到字符串 DONE 为止。消费者线程在消息读取时也会随机的暂停。

```java
import java.util.Random;

public class Consumer implements Runnable {
    private Drop drop;

    public Consumer(Drop drop) {
        this.drop = drop;
    }

    public void run() {
        Random random = new Random();
        for (String message = drop.take();
             ! message.equals("DONE");
             message = drop.take()) {
            System.out.format("MESSAGE RECEIVED: %s%n", message);
            try {
                Thread.sleep(random.nextInt(5000));
            } catch (InterruptedException e) {}
        }
    }
}
```

ProducerConsumerExample 是主线程，它启动生产者线程和消费者线程。

```java
public class ProducerConsumerExample {
    public static void main(String[] args) {
        Drop drop = new Drop();
        (new Thread(new Producer(drop))).start();
        (new Thread(new Consumer(drop))).start();
    }
}
```

#### 不可变对象

一个对象如果在创建后不能被修改，那么就称为不可变对象。

在并发编程中，一种被普遍认可的原则就是：尽可能的使用不可变对象来创建简单、可靠的代码。

在并发编程中，不可变对象特别有用。由于创建后不能被修改，所以不会出现由于线程干扰产生的错误或是内存一致性错误。

但是程序员们通常并不热衷于使用不可变对象，因为他们担心每次创建新对象的开销。实际上这种开销常常被过分高估，而且使用不可变对象所带来的一些效率提升也抵消了这种开销。

例如：使用不可变对象降低了垃圾回收所产生的额外开销，也减少了用来确保使用可变对象不出现并发错误的一些额外代码。

接下来看一个可变对象的类，然后转化为一个不可变对象的类。通过这个例子说明转化的原则以及使用不可变对象的好处。

#### 一个同步类的例子

SynchronizedRGB 是表示颜色的类，每一个对象代表一种颜色，使用三个整形数表示颜色的三基色，字符串表示颜色名称。

```java
public class SynchronizedRGB {

    // Values must be between 0 and 255.
    private int red;
    private int green;
    private int blue;
    private String name;

    private void check(int red, int green, int blue) {
        if (red < 0 || red > 255
            || green < 0 || green > 255
            || blue < 0 || blue > 255) {
            throw new IllegalArgumentException();
        }
    }

    public SynchronizedRGB(int red, int green, int blue, String name) {
        check(red, green, blue);
        this.red = red;
        this.green = green;
        this.blue = blue;
        this.name = name;
    }

    public void set(int red, int green, int blue, String name) {
        check(red, green, blue);
        synchronized (this) {
            this.red = red;
            this.green = green;
            this.blue = blue;
            this.name = name;
        }
    }

    public synchronized int getRGB() {
        return ((red << 16) | (green << 8) | blue);
    }

    public synchronized String getName() {
        return name;
    }

    public synchronized void invert() {
        red = 255 - red;
        green = 255 - green;
        blue = 255 - blue;
        name = "Inverse of " + name;
    }
}
```

使用 SynchronizedRGB 时需要小心，避免其处于不一致的状态。例如一个线程执行了以下代码：

```java
SynchronizedRGB color = new SynchronizedRGB(0, 0, 0, "Pitch Black");
...
int myColorInt = color.getRGB();      //Statement 1
String myColorName = color.getName(); //Statement 2
```

如果有另外一个线程在 Statement 1之后、Statement 2 之前调用了 color.set 方法，那么 myColorInt 的值和 myColorName 的值就会不匹配。为了避免出现这样的结果，必须要像下面这样把这两条语句绑定到一块执行：

```java
synchronized (color) {
    int myColorInt = color.getRGB();
    String myColorName = color.getName();
}
```

#### 定义不可变对象的策略

以下的一些规则是创建不可变对象的简单策略。并非所有不可变类都完全遵守这些规则，不过这不是编写这些类的程序员们粗心大意造成的，很可能的是他们有充分的理由确保这些对象在创建后不会被修改。但这需要非常复杂细致的分析，并不适用于初学者。

- 不要提供 setter 方法。（包括修改字段的方法和修改字段引用对象的方法）
- 将类的所有字段定义为 final、private 的。
- 不允许子类重写方法。简单的办法是将类声明为 final，更好的方法是将构造函数声明为私有的，通过工厂方法创建对象。
- 如果类的字段是对可变对象的引用，不允许修改被引用对象。
    - 不提供修改可变对象的方法。
    - 不共享可变对象的引用。当一个引用被当做参数传递给构造函数，而这个引用指向的是一个外部的可变对象时，一定不要保存这个引用。如果必须要保存，那么创建可变对象的拷贝，然后保存拷贝对象的引用。同样如果需要返回内部的可变对象时，不要返回可变对象本身，而是返回其拷贝。

将这一策略应用到 SynchronizedRGB 有以下几步：

- SynchronizedRGB 类有两个 setter 方法。第一个 set 方法只是简单的为字段设值（译者注：删掉即可），第二个 invert 方法修改为创建一个新对象，而不是在原有对象上修改。
- 所有的字段都已经是私有的，加上 final 即可。
- 将类声明为 final 的。
- 只有一个字段是对象引用，并且被引用的对象也是不可变对象。

```java
final public class ImmutableRGB {

    // Values must be between 0 and 255.
    final private int red;
    final private int green;
    final private int blue;
    final private String name;

    private void check(int red, int green, int blue) {
        if (red < 0 || red > 255
            || green < 0 || green > 255
            || blue < 0 || blue > 255) {
            throw new IllegalArgumentException();
        }
    }

    public ImmutableRGB(int red, int green, int blue, String name) {
        check(red, green, blue);
        this.red = red;
        this.green = green;
        this.blue = blue;
        this.name = name;
    }


    public int getRGB() {
        return ((red << 16) | (green << 8) | blue);
    }

    public String getName() {
        return name;
    }

    public ImmutableRGB invert() {
        return new ImmutableRGB(255 - red,
                       255 - green,
                       255 - blue,
                       "Inverse of " + name);
    }
}
```


【参考资料】

- [Oracle官方并发教程](http://docs.oracle.com/javase/tutorial/essential/concurrency/index.html)
- [Oracle官方并发教程译文](http://ifeve.com/oracle-guarded-blocks/)

---EOF---

