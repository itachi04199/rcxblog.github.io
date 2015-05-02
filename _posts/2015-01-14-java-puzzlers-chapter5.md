---
layout: post
title: Java 解惑 -- 类谜题
categories: Java解惑
tags: 笔记
---

#### 混淆的构造器

看如下代码会输出什么：

```java
public class Confusing {
    private Confusing(Object o) {
        System.out.println("Object");
    }
    private Confusing(double[] dArray) {
        System.out.println("double array");
    }
    public static void main(String[] args) {
        new Confusing(null);
    }
}
```

什么的代码会输出：double array。

Java 的重载解析过程是以两阶段运行的。第一阶段选取所有可获得并且可应用的方法或构造器。第二阶段在第一阶段选取的方法或构造器中选取最精确的一个。

Confusing(Object) 可以接受任何传递给 Confusing(double[ ]) 的参数，因此 Confusing(Object) 相对缺乏精确性。

要想用一个 null 参数来调用 Confusing(Object) 构造器，你需要这样写代码：new Confusing((Object)null)。这可以确保只有 Confusing(Objec t)是可应用的。更一般地讲，要想强制要求编译器选择一个精确的重载版本，需要将实际的参数转型为形式参数所声明的类型。

应该尽可能地避免重载，如果你必须进行重载，么请确保所有的重载版本所接受的参数类型都互不兼容。

#### 继承类中的静态域和方法

看下面代码的输出：

```java
class Counter {
    private static int count = 0;
    public static final synchronized void increment() {
        count++;
    }
    public static final synchronized int getCount() {
        return count;
    }
}

class Dog extends Counter {
    public Dog() { }
    public void woof() { increment(); }
}

class Cat extends Counter {
    public Cat() { }
    public void meow() { increment(); }
}

public class Ruckus {
    public static void main(String[] args) {
        Dog dogs[] = { new Dog(), new Dog() };
        for (int i = 0; i < dogs.length; i++)
            dogs[i].woof();
        Cat cats[] = { new Cat(), new Cat(), new Cat() };
        for (int i = 0; i < cats.length; i++)
            cats[i].meow();
        System.out.print(Dog.getCount() + " woofs and ");
        System.out.println(Cat.getCount() + " meows");
    }
}
```

它打印的是 5 woofs and 5 meows。

静态域由声明它的类及其所有子类所共享。如果你需要让每一个子类都具有某个域的单独拷贝，那么你必须在每一个子类中声明一个单独的静态域。如果每一个实例都需要一个单独的拷贝，那么你可以在基类中声明一个非静态域。

看下面代码的输出:

```java
class Dog {
    public static void bark() {
        System.out.print("woof ");
    }
}

class Basenji extends Dog {
    public static void bark() { }
}

public class Bark {
    public static void main(String args[]) {
        Dog woofer = new Dog();
        Dog nipper = new Basenji();
        woofer.bark();
        nipper.bark();
    }
}
```

打印的是 woof woof。

问题在于 bark 是一个静态方法，而对静态方法的调用不存在任何动态的分派机制。当一个程序调用了一个静态方法时，要被调用的方法都是在编译时刻被选定的，而这种选定是基于修饰符的编译期类型而做出的，修饰符的编译期类型就是我们给出的方法调用表达式中圆点左边部分的名字。

在本例中，两个方法调用的修饰符分别是变量 woofer 和 nipper，它们都被声明为 Dog 类型。

静态方法是不能被覆写的；它们只能被隐藏，而这仅仅是因为你没有表达出你应该表达的意思。为了避免这样的混乱，千万不要隐藏静态方法。

#### 静态初始化时机

看下面程序：

```java
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private final int beltSize;
    private static final int CURRENT_YEAR =
        Calendar.getInstance().get(Calendar.YEAR);
    private Elvis() {
        beltSize = CURRENT_YEAR - 1930;
    }
    public int beltSize() {
        return beltSize;
    }
    public static void main(String[] args) {
        System.out.println("Elvis wears a size " +
                           INSTANCE.beltSize() + " belt.");
    }
}
```

该程序将打印出 Elvis wears a size -1930 belt。

Elvis 类的初始化是由虚拟机对其 main 方法的调用而触发的。首先，其静态域被设置为缺省值，其中 INSTANCE 域被设置为 null， CURRENT_YEAR 被设置为 0。接下来，静态域初始器按照其出现的顺序执行。第一个静态域是 INSTANCE，它的值是通过调用 Elvis() 构造器而计算出来的。

这个构造器会用一个涉及静态域 CURRENT_YEAR 的表达式来初始化 beltSize。通常，读取一个静态域是会引起一个类被初始化的事件之一，但是我们已经在初始化 Elvis 类了。递归的初始化尝试会直接被忽略掉。因此，CURRENT_YEAR 的值仍旧是其缺省值 0。这就是为什么 Elvis 变成了-1930的原因。

#### 类型转换

看下面三个例子输出是什么：

```java
public class Type1 {
    public static void main(String[] args) {
        String s = null;
        System.out.println(s instanceof String);
    }
}

public class Type2 {
    public static void main(String[] args) {
        System.out.println(new Type2() instanceof String);
    }
}

public class Type3 {
    public static void main(String args[]) {
        Type3 t3 = (Type3) new Object();
    }
}
```

第一个程序，Type1，展示了 instanceof 操作符应用于一个空对象引用时的行为。尽管 null 对于每一个引用类型来说都是其子类型，但是 instanceof 操作符被定义为在其左操作数为 null 时返回 false。因此，Type1 将打印 false。

如果 instanceof 告诉你一个对象引用是某个特定类型的实例，那么你就可以将其转型为该类型，并调用该类型的方法，而不用担心会抛出 ClassCastException 或 NullPointerException 异常。

第二个程序，Type2，展示了 instanceof 操作符在测试一个类的实例，以查看它是否是某个不相关的类的实例时所表现出来的行为。你可能会期望该程序打印出 false。但是这个程序会在编译时候报错， 因为 instanceof 操作符有这样的要求：如果两个操作数的类型都是类，其中一个必须是另一个的子类型。

第三个程序，Type3，展示了当要被转型的表达式的静态类型是转型类型的超类时，转型操作符的行为。与 instanceof 操作相同，如果在一个转型操作中的两种类型都是类，那么其中一个必须是另一个的子类型。该程序将在运行期抛出 ClassCastException 异常，但是却能编译。

#### 继承类初始化

看下面程序打印什么：

```java
class Point {
    protected final int x, y;
    private final String name; // Cached at construction time
    Point(int x, int y) {
        this.x = x;
        this.y = y;
        name = makeName();//step3
    }

    protected String makeName() {
        return "[" + x + "," + y + "]";
    }
    public final String toString() {
        return name;
    }
}

public class ColorPoint extends Point {
    private final String color;
    ColorPoint(int x, int y, String color) {
        super(x, y);//step2
        this.color = color;//step5
    }
    protected String makeName() {
       return super.makeName() + ":" + color;//step4
    }
    public static void main(String[] args) {
        System.out.println(new ColorPoint(4, 2, "purple"));//step1
    }
}
```

运行程序它打印的是 [4,2]:null。

这个程序帮助我们理解下实例初始化顺序。

首先，程序通过调用 ColorPoint 构造器 step1。这个构造器调用其超类构造器 step2。然后该超类构造器调用 makeName，该方法被子类覆写了 step3。ColorPoint 中的 makeName方法 step4。makeName 方法首先调用 super.makeName，它将返回我们所期望的[4,2]，然后该方法在此基础上追加字符串 “：” 和由 color 域的值所转换成的字符串。但是此刻 color 域仍处于待初始化状态，所以它的值仍旧是缺省值null。因此，makeName方法返回的是字符串 “[4,2]:null”。

要想避免这个问题，就千万不要在构造器中调用可覆写的方法，直接调用或间接调用都不行。

#### 静态块初始化顺序

看下面程序打印：

```java
class Cache {
    static {
        initializeIfNecessary();
    }
    private static int sum;
    public static int getSum() {
        initializeIfNecessary();
        return sum;
    }

    private static boolean initialized = false;
    private static synchronized void initializeIfNecessary() {
        if (!initialized) {
            for (int i = 0; i < 100; i++)
                sum += i;
            initialized = true;
        }
    }
}
public class Client {
    public static void main(String[] args) {
        System.out.println(Cache.getSum());
    }
}
```

这个程序会打印出 9900。

Cache 类有两个静态初始器：在类顶端的一个 static 语句块，以及静态域 initialized 的初始化。静态语句块是先出现的，它调用了方法 initializeIfNecessary，该方法将测试 initialized域。因为该域还没有被赋予任何值，所以它具有缺省的布尔值false。与此类似，sum 具有缺省的 int 值 0。因此，initializeIfNecessary 方法执行的正是你所期望的行为，将 4,950 添加到了 sum 上，并将 initialized 设置为true。

在静态语句块执行之后，initialized 域的静态初始器将其设置回 false，从而完成 Cache 的类初始化。遗憾的是，sum 现在包含的是正确的缓存值，但是 initialized 包含的却是 false。

此后，Client 类的 main 方法调用 Cache.getSum 方法，它将再次调用 initializeIfNecessary 方法。因为 initialized 标志是 false，所以 initializeIfNecessary 方法将进入其循环，该循环将把另一个 4,950 添加到 sum 上，从而使其值增加到了 9,900。

如果要修复上面的问题，可以将代码修改成如下：

```java
class Cache {
    private static final int sum = computeSum();
    private static int computeSum() {
        int result = 0;
        for (int i = 0; i < 100; i++)
            result += i;
        return result;
    }
    public static int getSum() {
        return sum;
    }
}
```

我们使用了一个助手方法来初始化 sum。助手方法通常都优于静态语句块，因为它让你可以对计算命名。只有在极少的情况下，你才必须使用一个静态语句块来初始化一个静态域，此时请将该语句块紧随该域声明之后放置。这提高了程序的清晰度，并且消除了像最初的程序中出现的静态初始化与静态语句块互相竞争的可能性。

#### Null 与 Void

看下面程序打印：

```java
public class Null {
    public static void greet() {
        System.out.println("Hello world!");
    }
    public static void main(String[] args) {
        ((Null) null).greet();
    }
}
```

程序会打印出 Hello world!。

在调用静态方法时候，使用表达式作为其限定符并非是一个好主意，不仅表达式的值所引用的对象的运行期类型在确定哪一个方法将被调用时并不起任何作用，而且如果对象有标识的话，其标识也不起任何作用。在本例中，没有任何对象，但是这并不会造成任何区别。静态方法调用的限定表达式是可以计算的，但是它的值将被忽略。没有任何要求其值为非空的限制	。

#### 特别创建对象

看下面程序：

```java
public class Creator {
    public static void main(String[] args) {
        for (int i = 0; i < 100; i++)
            Creature creature = new Creature();
        System.out.println(Creature.numCreated());
    }
}

class Creature {
    private static long numCreated = 0;
    public Creature() {
        numCreated++;
    }
    public static long numCreated() {
        return numCreated;
    }
}
```

上面这段程序编译时候会出现问题，如下：

```java
Creator.java:4: not a statement
            Creature creature = new Creature();
            ^
Creator.java:4: ';' expected
            Creature creature = new Creature();
                            ^
```

一个本地变量声明看起来像是一条语句，但是从技术上说，它不是；它应该是一个本地变量声明语句（local variable declaration statement）[JLS 14.4]。Java语言规范不允许一个本地变量声明语句作为一条语句在 for、while 或 do 循环中重复执行 [JLS 14.12-14]。一个本地变量声明作为一条语句只能直接出现在一个语句块中。

上面这段程序的修改就可以使用：

```java
        for (int i = 0; i < 100; i++) {
            Creature creature = new Creature();
        }
        for (int i = 0; i < 100; i++)
            new Creature();
```

【参考资料】

1.  [java解惑](http://book.douban.com/subject/1473329/)

---EOF---
