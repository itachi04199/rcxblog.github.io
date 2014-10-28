---
layout: post
title: Java内部类
categories: java基础
tags: 笔记
---

#### 为什么Java支持嵌套类

Java支持嵌套类有两个好处：

- 让代码变得更清晰
- 减少名字冲突

使用内部类可以使代码更清晰，是因为你可以定义一个封闭的类，然后它可以方便的操纵这些对象的属性和调用这些对象的方法，甚至是私有的属性和方法。

考虑下面的例子会更加清晰，程序遍历Employee对象当中的Job数组对象。

```java
class Job
{
   private String jobTitle;
   Job (String jobTitle)
   {
      this.jobTitle = jobTitle;
   }
   public String toString ()
   {
      return jobTitle;
   }
}

class Employee
{
   private String name;
   private Job [] jobs;
   private int jobIndex = 0;
   Employee (String name, Job [] jobs)
   {
      this.name = name;
      this.jobs = jobs;
   }
   String getName ()
   {
      return name;
   }
   boolean hasMoreJobs ()
   {
      return jobIndex < jobs.length;
   }
   Job nextJob ()
   {
      return !hasMoreJobs () ? null : jobs [jobIndex++];
   }
}

class JobIterator1
{
   public static void main (String [] args)
   {
      Job [] jobs = { new Job ("Janitor"), new Job ("Delivery Person") };
      Employee e = new Employee ("John Doe", jobs);
      System.out.println (e.getName () + " works the following jobs:\n");
      while (e.hasMoreJobs ())
         System.out.println (e.nextJob ());
   }
}
```

运行程序输出如下：

```
John Doe works the following jobs:
Janitor
Delivery Person
```

看Employee当中的 hasMoreJobs() 和 nextJob() 方法，这两个方法相当于创建了一个迭代器。

JobIterator1有几个问题。

- 你不能重新进行迭代。当然你可以给Employee类增加一个reset()方法，当迭代结束给jobIndex设置成为0。
- 更严重的问题是同一个Employee对象不可以创建多个迭代器。这个问题是因为hasMoreJobs() 和 nextJob() 方法在Employee类内部。

为了解决上面的两个问题，看下面的例子：

```java
class Job
{
   private String jobTitle;
   Job (String jobTitle)
   {
      this.jobTitle = jobTitle;
   }
   public String toString ()
   {
      return jobTitle;
   }
}

class Employee
{
   private String name;
   private Job [] jobs;
   Employee (String name, Job [] jobs)
   {
      this.name = name;
      this.jobs = jobs;
   }
   String getName ()
   {
      return name;
   }
   JobIterator getJobIterator ()
   {
      return new JobIterator (jobs);
   }
}

class JobIterator
{
   private Job [] jobs;
   private int jobIndex = 0;
   JobIterator (Job [] jobs)
   {
      this.jobs = jobs;
   }
   boolean hasMoreJobs ()
   {
      return jobIndex < jobs.length;
   }
   Job nextJob ()
   {
      return !hasMoreJobs () ? null : jobs [jobIndex++];
   }
}

class JobIterator2
{
   public static void main (String [] args)
   {
      Job [] jobs = { new Job ("Janitor"), new Job ("Delivery Person") };
      Employee e = new Employee ("John Doe", jobs);
      System.out.println (e.getName () + " works the following jobs:\n");
      JobIterator ji = e.getJobIterator ();
      while (ji.hasMoreJobs ())
         System.out.println (ji.nextJob ());
   }
}
```

运行结果与JobIterator1的输出一样，JobIterator2与JobIterator1的代码区别就是把迭代器的代码从Employee中移动到了JobIterator类当中。尽管，Employee类定义了一个getJobIterator()方法，但是它每次返回一个新的JobIterator 对象。

Employee和JobIterator类是紧密耦合的，JobIterator的构造函数需要Employee的私有变量jobs。

尽管解决了JobIterator1的问题，但是引入了新的问题：新添加的类JobIterator是和Employee处于同一级别，如果将来增加JobIterator接口会引起文件名重复的冲突。在我们例子当中这个问题不是很大，但是在很大的项目当中就应当考虑这种情况。为了能让名字共存，一些类就需要完全的依赖其他的类。

定义内部类来解决这样的问题，例子如下：

```java
class Job
{
   private String jobTitle;
   Job (String jobTitle)
   {
      this.jobTitle = jobTitle;
   }
   public String toString ()
   {
      return jobTitle;
   }
}

class Employee
{
   private String name;
   private Job [] jobs;
   Employee (String name, Job [] jobs)
   {
      this.name = name;
      this.jobs = jobs;
   }
   String getName ()
   {
      return name;
   }
   JobIterator getJobIterator ()
   {
      return new JobIterator ();
   }
   class JobIterator
   {
      private int jobIndex = 0;
      public boolean hasMoreJobs ()
      {
         return jobIndex < jobs.length;
      }
      public Object nextJob ()
      {
         return !hasMoreJobs () ? null : jobs [jobIndex++];
      }
   }
}

class JobIterator3
{
   public static void main (String [] args)
   {
      Job [] jobs = { new Job ("Janitor"), new Job ("Delivery Person") };
      Employee e = new Employee ("John Doe", jobs);
      System.out.println (e.getName () + " works the following jobs:\n");
      Employee.JobIterator eji = e.getJobIterator ();
      while (eji.hasMoreJobs ())
         System.out.println (eji.nextJob ());
   }
}
```

JobIterator3的输出结果与JobIterator2和JobIterator1是一样的。嵌套类JobIterator不需要那个构造函数，因为它可以直接访问外部类的私有变量。这个改动使代码更加清晰了，同时减少了名字的冲突。

#### Java支持什么样的嵌套类

Java支持两种嵌套类：(静态内部类)nested top-level classes 和 (内部类)inner classes。

此外，(内部类)inner classes分为，(成员内部类) instance inner class, (局部内部类) local inner class, and (匿名内部类) anonymous inner class。

#### 静态内部类 (nested top-level classes)

在类的内部声明一个类，并且使用static关键字进行修饰的类就是静态内部类。

```java
class TopLevelClass
{
   static class NestedTopLevelClass
   {
   }
}
```

就像一个静态字段和静态方法是独立于任何对象，他们是属于类的。考虑下面的代码片段：

```java
class TopLevelClass
{
   static int staticField;
   int instanceField;
   static class NestedTopLevelClass
   {
      static
      {
         System.out.println ("Can access staticField " + staticField);
//         System.out.println ("Cannot access instanceField " + instanceField);
      }
      {
         System.out.println ("Can access staticField " + staticField);
//         System.out.println ("Cannot access instanceField " + instanceField);
      }
   }
}
```

从上面的例子可以看出NestedTopLevelClass可以访问TopLevelClass的静态变量，但是不可以访问TopLevelClass的实例变量。因为NestedTopLevelClass是独立于任何TopLevelClass对象的。

> 一个静态内部类不能访问任何外部类的实例成员(属性和方法)。

尽管静态内部类不能访问任何外部类的实例成员，但是它自己可以声明实例变量。创建NestedTopLevelClass实例如下：

```java
class TopLevelClass
{
   static class NestedTopLevelClass
   {
      int myInstanceField;
      NestedTopLevelClass (int i)
      {
         myInstanceField = i;
      }
   }
}

class NestedTopLevelClassDemo
{
   public static void main (String [] args)
   {
      TopLevelClass.NestedTopLevelClass ntlc;
      ntlc = new TopLevelClass.NestedTopLevelClass (5);
      System.out.println (ntlc.myInstanceField);
   }
}
```

运行后输出的结果为：5。

你是否想知道，静态内部类是否还可以包含静态内部类。并且，如果有两个相同名字的静态属性会如何处理，看下面的例子：

```java
class TopLevelClass
{
   private static int a = 1;
   private static int b = 3;
   static class NestedTopLevelClass
   {
      private static int a = 2;
      static class NestedNestedTopLevelClass
      {
         void printFields ()
         {
            System.out.println ("a = " + a);
            System.out.println ("b = " + b);
         }
      }
   }
}

class NestingAndShadowingDemo
{
   public static void main (String [] args)
   {
      TopLevelClass.NestedTopLevelClass.NestedNestedTopLevelClass nntlc;
      nntlc = new TopLevelClass.NestedTopLevelClass.NestedNestedTopLevelClass ();
      nntlc.printFields ();
   }
}
```

运行结果如下：

```
a = 2
b = 3
```

> 你可以使用private,protected,public,(包括无关键字)关键字来修饰静态内部类。

#### 成员内部类 (Instance inner classes)

成员内部类可以访问静态和实例成员。像JobIterator3这个例子当中的JobIterator类就是成员内部类。

> 你可以使用private,protected,public,(包括无关键字)关键字来修饰成员内部类。

#### 局部内部类 (Local inner classes)

嵌套类不仅仅可以在类的内部声明，Java也运行在任何代码块当中声明内部类。这表示可以出现在方法当中，甚至是if语句当中。这样的嵌套类称之为局部内部类。

局部内部类比成员内部类有一个优势，不仅仅可以访问外部类的实例变量和方法，还可以访问局部的变量或者方法的参数，看下面的例子：

```java
class ComputerLanguage
{
   private String name;
   ComputerLanguage (String name)
   {
      this.name = name;
   }
   public String toString ()
   {
      return name;
   }
}

class LocalInnerClassDemo
{
   public static void main (String [] args)
   {
      ComputerLanguage [] cl = 
      {
         new ComputerLanguage ("Ada"),
         new ComputerLanguage ("Algol"),
         new ComputerLanguage ("APL"),
         new ComputerLanguage ("Assembly - IBM 360"),
         new ComputerLanguage ("Assembly - Intel"),
         new ComputerLanguage ("Assembly - Mostek"),
         new ComputerLanguage ("Assembly - Motorola"),
         new ComputerLanguage ("Assembly - VAX"),
         new ComputerLanguage ("Assembly - Zilog"),
         new ComputerLanguage ("BASIC"),
         new ComputerLanguage ("C"),
         new ComputerLanguage ("C++"),
         new ComputerLanguage ("Cobol"),
         new ComputerLanguage ("Forth"),
         new ComputerLanguage ("Fortran"),
         new ComputerLanguage ("Java"),
         new ComputerLanguage ("LISP"),
         new ComputerLanguage ("Logo"),
         new ComputerLanguage ("Modula 2"),
         new ComputerLanguage ("Pascal"),
         new ComputerLanguage ("Perl"),
         new ComputerLanguage ("Prolog"),
         new ComputerLanguage ("Snobol")
      };
      Enumeration e = enumerator ((Object []) cl);
      while (e.hasMoreElements ())
         System.out.println (e.nextElement ());
   }

   static Enumeration enumerator (final Object [] array)
   {
      class LocalInnerClass implements Enumeration
      {
         private int index = 0;
         public boolean hasMoreElements ()
         {
            return index < array.length;
         }
         public Object nextElement ()
         {
            return array [index++].toString ();
         }
      }
      return new LocalInnerClass ();
   }
```

可以看到局部内部类访问了方法的array的参数，并且这个参数的定义必须是final的。

#### 匿名内部类 (Anonymous inner classes)

当一个类很短，并且你想要声明一个没有名字的局部内部类，这个类就是匿名内部类。

示例代码如下：

```java
abstract class Farmer
{
   protected String name;
   Farmer (String name)
   {
      this.name = name;
   }
   abstract void occupation ();
}

class BeefFarmer extends Farmer
{
   BeefFarmer (String name)
   {
      super (name);
   }
   void occupation ()
   {
      System.out.println ("Farmer " + name + " raises beef cattle");
   }
}

class AnonymousInnerClassDemo1
{
   public static void main (String [] args)
   {
      BeefFarmer bf = new BeefFarmer ("John Doe");
      bf.occupation ();
      new Farmer ("Jane Doe")
          {
              void occupation ()
              {
                 System.out.println ("Farmer " + name + " milks cows");
              }
          }.occupation ();
   }
}
```

运行后输出如下：

```
Farmer John Doe raises beef cattle
Farmer Jane Doe milks cows
```

匿名内部类使用父类的构造函数来进行初始化对象。我们是否可以使用自己定义的构造函数在匿名内部类中，答案是否定的，因为构造函数需要类名，而匿名内部类没有。

但是可以使用代码块来初始化对象，当创建匿名内部类的时候。

看下面的例子：

```java
abstract class Farmer
{
   protected String name;
   Farmer (String name)
   {
      this.name = name;
   }
   abstract void occupation ();
}

class BeefFarmer extends Farmer
{
   BeefFarmer (String name)
   {
      super (name);
   }
   void occupation ()
   {
      System.out.println ("Farmer " + name + " raises beef cattle");
   }
}

class AnonymousInnerClassDemo2
{
   public static void main (final String [] args)
   {
      BeefFarmer bf = new BeefFarmer ("John Doe");
      bf.occupation ();
      new Farmer ("Jane Doe")
          {
              private String count;

              {
                 if (args.length == 1)
                     count = args [0];
              }

              void occupation ()
              {
                 if (count == null)
                     System.out.println ("Farmer " + name + " milks cows");
                 else
                     System.out.println ("Farmer " + name + " milks " +
                                         count + " cows");
              }
          }.occupation ();
   }
}
```

如果运行上面的例子，并且在命令行传入参数10，则输出如下：

```
Farmer John Doe raises beef cattle
Farmer Jane Doe milks 10 cows
```

> 即使匿名内部类没有名字，在编译的时候仍然会产生一个类文件。产生的类名是外部类+$+一个整数。上面例子中不但会生成一个AnonymousInnerClassDemo2.class文件，还会生成一个AnonymousInnerClassDemo2$1.class文件。

考虑一下匿名内部类的实际应用，通常匿名内部类用于简化事件处理。看下面的例子：

```java
class AnonymousInnerClassDemo3
{
   public static void main (String [] args)
   {
      Frame f = new Frame ("Anonymous Inner Class Demo #3");
      f.addWindowListener (new WindowAdapter ()
                           {
                               public void windowClosing (WindowEvent e)
                               {
                                  System.exit (0);
                               }
                           });
      f.setSize (300, 100);
      f.setVisible (true);
   }
}
```

注册一个监听器，用于处理点击叉关闭整个界面的处理工作。

#### 回顾

Java有四种嵌套类：

- 静态内部类
- 成员内部类
- 局部内部类
- 匿名内部类

相比之下，成员内部类可以访问外部类的成员变量和方法。而静态内部类只可以访问外部类的静态变量和静态方法。

Java支持局部内部类出现在任何代码块当中，包括一个方法中或者if语句当中。此外，由于一些局部内部类很短并且不需要名字，所以有了匿名内部类。局部内部类和匿名内部类可以访问final的局部变量和方法参数。

【参考资料】

1. [Classes within classes](http://www.javaworld.com/article/2074000/core-java/classes-within-classes.html?page=1)



---EOF---

