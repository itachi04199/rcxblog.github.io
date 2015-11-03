---
layout: post
title: Java Executor 框架学习
categories: java基础
tags: 笔记, java
---

大多数并发都是通过任务执行的方式来实现的。一般有两种方式执行任务：串行和并行。

```java
class SingleThreadWebServer {
	public static void main(String[] args) throws Exception {
    	ServerSocket socket = new ServerSocket(80);
        while(true) {
        	Socket conn = socket.accept();
            handleRequest(conn);
        }
    }
}

class ThreadPerTaskWebServer {
	public static void main(String[] args) throws Exception {
    	ServerSocket socket = new ServerSocket(80);
        while(true) {
        	final Socket conn = socket.accept();
            Runnable task = new Runnable() {
            	public void run() {
                	handleRequest(conn);
                }
            };
            new Thread(task).start();
        }
    }
}
```

当然上面的这两种方式都是有问题的。单线程的问题就是并发量会是瓶颈，多线程版本就是无限制的创建线程会导致资源不足问题。

## Executor 框架

任务是一组逻辑工作单元，而线程是使任务异步执行的机制。

JDK 提供了 Executor 接口：

```java
public interface Executor {
    void execute(Runnable command);
}
```

虽然 Executor 接口比较简单，但是却是异步任务执行框架的基础，该框架能支持多种不同类型的任务执行策略。它提供了一种标准的方式把任务的提交过程与执行过程进行了解耦。用 Runnable 来代表任务。Executor 的实现提供了对生命周期的支持以及统计信息应用程序管理等机制。

Executor 是基于生产者消费者模式的，提交任务的操作相当于生产者，执行任务的线程相当于消费。

基于 Executor 的 WebServer 例子如下：

```java
public class TaskExecutorWebServer {
	private static final int NTHREADS = 100;
	private static final Executor exec = Executors.newFixedThreadPool(NTHREADS);

	public static void main(String[] args) throws Exception {
		ServerSocket serverSocket = new ServerSocket(80);
		while (true) {
			final Socket conn = serverSocket.accept();
			Runnable task = new Runnable() {
				@Override
				public void run() {
					handleRequest(conn);
				}
			};
			exec.execute(task);
		}
	}
}
```

另外可以自己实现 Executor 来控制是并发还是并行的，如下面代码：

```java
/**
 * 执行已提交的 Runnable 任务的对象。
 * 此接口提供一种将任务提交与每个任务将如何运行的机制（包括线程使用的细节、调度等）分离开来的方法。
 * 通常使用 Executor 而不是显式地创建线程。
 *
 *
 * @author renchunxiao
 *
 */
public class ExecutorDemo {
	public static void main(String[] args) {
		Executor executor = new ThreadExecutor();
		executor.execute(new Runnable() {
			@Override
			public void run() {
				// do something
			}
		});

		Executor executor2 = new SerialExecutor();
		executor2.execute(new Runnable() {
			@Override
			public void run() {
				// do something
			}
		});

	}
}

/**
 * 创建一个线程来执行 command
 *
 * @author renchunxiao
 *
 */
class ThreadExecutor implements Executor {
	@Override
	public void execute(Runnable command) {
		new Thread(command).start();
	}
}

/**
 * 串行执行 command
 *
 * @author renchunxiao
 *
 */
class SerialExecutor implements Executor {
	@Override
	public void execute(Runnable command) {
		command.run();
	}
}
```

## 线程池

线程池就是线程的资源池，可以通过 Executors 中的静态工厂方法来创建线程池。

- newFixedThreadPool。创建固定长度的线程池，每次提交任务创建一个线程，直到达到线程池的最大数量，线程池的大小不再变化。
- newSingleThreadExecutor。单个线程池。
- newCachedThreadPool。根据任务规模变动的线程池。
- newScheduledThreadPool。创建固定长度的线程池，以延迟或定时的方式来执行任务。

JVM 只有在所有非守护线程全部终止后才会退出，所以，如果无法正确的关闭 Executor，那么 JVM 就无法结束。

为了解决执行服务的生命周期问题，有个扩展 Executor 接口的新接口 ExecutorService。

```java
public interface ExecutorService extends Executor {

    void shutdown();

    List<Runnable> shutdownNow();

    boolean isShutdown();

    boolean isTerminated();

    boolean awaitTermination(long timeout, TimeUnit unit)
        throws InterruptedException;

    <T> Future<T> submit(Callable<T> task);

	<T> Future<T> submit(Runnable task, T result);

    Future<?> submit(Runnable task);

    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks)
        throws InterruptedException;

    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks,
                                  long timeout, TimeUnit unit)
        throws InterruptedException;

    <T> T invokeAny(Collection<? extends Callable<T>> tasks)
        throws InterruptedException, ExecutionException;

    <T> T invokeAny(Collection<? extends Callable<T>> tasks,
                    long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```

ExecutorService 生命周期有三种状态：运行、关闭、已终止。ExecutorService 在初始创建时处于运行状态。shutdown 方法会平缓关闭：不在接受新的任务，并且等待已经执行的任务执行完成(包括那些还未开始的任务)。shutdownNow 方法将粗暴关闭：它将尝试取消所有运行中的任务，并且不再启动队列中尚未开始的任务。所有任务都执行完成后进入到已终止状态。

## Callable 和 Future

Executor 框架使用 Runnable 作为基本的任务表示形式。Runnable 是一种有局限性的抽象，它的 run 方法不能返回值和抛出一个受检查异常。

许多任务实际上是存在延时的计算，例如数据库查询，从网络获取资源。对于这些任务，Callable 是更好的抽象，它认为 call 将返回一个值，并且可能抛出异常。

Executor 执行的任务有四个生命周期阶段:创建、提交、开始和完成。由于有些任务需要很长时间有可能希望取消，在 Executor 框架当中，已提交未开始的任务可以取消。

Future 表示一个任务的生命周期，并且提供了相应的方法来判断是否已经完成或取消，以及获取任务的结果和取消任务等。

【参考资料】

1. [Java并发编程实战](http://book.douban.com/subject/10484692/)

---EOF---
