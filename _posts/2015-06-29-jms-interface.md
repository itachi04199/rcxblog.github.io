---
layout: post
title: JMS 入门学习之概念及接口
categories: jms
tags: java, jms
---

## JMS 概念

JMS 有两种方式进行消息通信：Point-to-Point (P2P) 和 Publish/Subscriber (PUB/SUB)

- JMS（Java Messaging Service）
是Java EE中的一种技术，它定义一套完整的接口，来实现不同系统或应用之间的消息通信。这意味着：我们针对 JMS 接口编写的应用程序（客户程序），在任何一个实现了标准 JMS 接口的容器下都能运行起来，我们的应用程序与容器实现了真正的解藕，这也就是面向接口编程的好处之一吧。这点类似 JDBC 编程。

- JMS 提供者（JMS Provider）
JMS 提供者，也叫 JMS 服务器或 JMS 容器，也就是 JMS 服务的提供者，主流的 J2EE 容器一般都提供 JMS 服务（比如JBoss，BEA WebLogic，IBM WebSphere，Oracle Application Server等都支持）

- 连接工厂（Connection Factories）
连接工厂是用来创建客户端到 JMS 容器之间 JMS 连接的工厂，连接工厂有两种：(QueueConnectionFactory 和 TopicConnectionFactory)，分别用来创建QueueConnection 和 TopicConnection的。

```java
Context ctx = new InitialContext();
QueueConnectionFactory queueConnectionFactory = 
                    (QueueConnectionFactory) ctx.lookup("QueueConnectionFactory");
TopicConnectionFactory topicConnectionFactory = 
                    (TopicConnectionFactory) ctx.lookup("TopicConnectionFactory");
```

- 目的地（Destinations）
目的地是消息生产者(producer)消息发住的目的地，也是消费者(consumer)接收消息的来源地，它有点像信箱，邮递员把信件投往信箱，收件人从信箱取信件。对 P2P 方式来说，目的地就是 Queue，对 pub/sub 方式来说，目的地就是 Topic。我们要得到这个目的地的引用，只能通过 JNDI 查找(lookup)的方式得到，因为目的地是注册在 JMS 服务器的（后面的章节会讲到如何注册一个目的地）

```java
Topic myTopic = (Topic) ctx.lookup("MyTopic");
Queue myQueue = (Queue) ctx.lookup("MyQueue");
```

- 连接（Connection）
这里说的连接是指客户端与JMS提供者（容器）之间的连接。连接也分两种：QueueConnection 和 TopicConnection，分别对应于 P2P 连接和 Pub/Sub 连接。

```java
QueueConnection queueConnection = queueConnectionFactory.createQueueConnection();
TopicConnection topicConnection = topicConnectionFactory.createTopicConnection();
```

连接用完之后必须记得关闭，否则连接资源不会被释放掉。关闭连接的同时会自动把会话、产生者、消费者都关闭掉。

- 会话（Session）
会话是用来创建消息产生者和消息消费者的单线程环境，你可以它来创建消息生产者、消费者、消息，用它来维持消息监听。

```java
TopicSession topicSession = topicConnection.createTopicSession(false, Session.AUTO_ACKNOWLEDGE);
QueueSession queueSession = queueConnection.createQueueSession(true, 0);
```

- 消息生产者（Message Producers）
消息生产者也就是消息的产生者或发送者，在 P2P 方式下它是 QueueSender，在 Pub/Sub 方式下它是 TopicPublisher。它是一个由 session 创建的，用来把把消息发送到目的地的对象。

```java
QueueSender queueSender = queueSession.createSender(myQueue);
TopicPublisher topicPublisher = topicSession.createPublisher(myTopic);
//一旦你创建好生产者，你就可以用它来发送消息
queueSender.send(message);
topicPublisher.publish(message);
```

- 消息消费者（Message Consumer）
消息消费者也就是消息的接收者或使用者，在 P2P 方式下这是 QueueReceiver，在 Pub/Sub 方式下它是 TopicSubscriber。这是一个由 session 来创建的，用来接收来自目的地消息的对象。JMS 容器来负责把消息从目的地投递到注册了该目的地的消息消费者。

```java
QueueReceiver queueReceiver = queueSession.createReceiver(myQueue);
TopicSubscriber topicSubscriber = topicSession.createSubscriber(myTopic);
```

一旦创建好消息消费者，它就是活动的，你可以用它来接收消息，你也可以用 close() 方法来使它失效（Inactive）。当你调用 Connection 的 start() 方法之前，消费者是不会接收到任何消息的。两种接收者都有一个 receive 方法，这是一个同步的方法，也就是说程序执行到这个方法会被阻塞，直到收到消息为止。

```java
queueConnection.start();
Message m = queueReceiver.receive();
topicConnection.start();
Message m = topicSubscriber.receive(1000); // time out after a second
```

如果我们不想它被阻塞，就需要异步的接收消息，这时我们得用消息临听器（Message Listener）了。

-  消息监听器（Message Listener）
消息监听器是一个充当消息的异步事件处理器的对象，它实现了 MessageListener 接口，这个接口只有一个方法 onMessage，在这个方法里，你可以定义当接收到消息之后的要做的操作。你可以用 setMessageListener 方法为消息消费者注册一个监听器。

```java
MessageListener listener = new MessageListener( {
      public void onMessage(Message msg) {
          //
      }
});
topicSubscriber.setMessageListener(listener); //注册监听
topicConnection.start();
```

有一点要注意，如果你先调用 Connection 的 start，然后再调用 setMessageListener，消息很可能接收不到，正确的做法是先注册监听，再启动 Connection。

注册监听之后，一旦 JMS 容器有消费投递过来，消息消费（接收）者就会自动调用监听器的 onMessage 方法。这个方法的带有一个参数 Message，这就接收到的消息。

## JMS 主要接口

- ConnectionFactory
- Destination
- Connection
- Session
- Message
- MessageProducer
- MessageConsumer

ConnectionFactory  和 Destination 必须使用 JNDI 从提供者处获得。其他接口可以用过工厂方法在不同的 API 接口中创建。

ConnectionFactory 可以创建一个 Connection。Connection 可以创建一个 Session。Session 可以创建一个 Message、MessageProducer、MessageConsumer。

![jms 公共 api](http://renchx.com/public/images/jms1.png)

![jms 不同通信方式](http://renchx.com/public/images/jms2.png)

【参考资料】

1. [java消息服务](http://book.douban.com/subject/4210586/)
2. [jms入门博客](http://www.cnblogs.com/jjj250/archive/2012/08/08/2628552.html)

---EOF---
