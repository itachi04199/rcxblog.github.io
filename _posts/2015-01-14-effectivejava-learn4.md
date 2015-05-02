---
layout: post
title: effective-java类和接口2
categories: effective-java
tags: 笔记
---

#### 接口优于抽象类

现有的类可以很容易被更新，以实现新的接口。

接口是定义 mixin 混合类型的理想选择。例如，Comparable 是一个 mixin 接口。

接口允许我们构造非层次结构的类型框架。

虽然接口不允许包含方法的实现，但是，可以对每个重要的接口都提供一个抽象的骨架实现类。

例如：AbstractMap、AbstractCollection 等都是属于骨架抽象类。

抽象类的演变比接口的演变容易的多。在抽象类中增加新的方法，始终可以增加具体的方法，它包含合理的默认实现。对于接口，这样是不行的。在设计接口的时候要非常谨慎。

#### 接口只能用于定义类型

有一种接口是常量接口，接口里面定义的全是常量，并且不包含任何方法。在使用的时候类来实现这个接口，就可以使用这个接口当中定义的常量。这种使用方式是对接口的不良使用。如果非 final 类实现了常量接口，那么它的所有子类都会包含这些常量。

总之，接口应该只定义类型，不应该被用来导出常量。

#### 用函数对象表示策略

在其他语言当中可以在方法当中传递函数。Java 没有提供这个功能，但是我们可以传递对象。如 Comparator 接口就是一个策略接口，当我们给 list 进行排序的时候，如果想要获得不同的结果可以传递不同的 Comparator 对象。

简单的说，函数指针主要用途就是实现策略模式。在 Java 当中实现这种模式，要声明一个接口来表示该策略，并且为每个具体策略声明一个实现了该接口的类。

#### 优先考虑静态成员类

Java 支持嵌套类，具体看[这篇文章](http://renchx.com/java-inner-class/)。

静态成员类是最简单的一种嵌套类。仅当与它的外部类一起使用时候才有意义。例如，考虑一个枚举，他描述计算器的各种操作。Operation 枚举应该是 Caculator 类的公有静态成员类，使用的时候 Caculator.Operation.PLUS 这样。

非静态成员类，一种常见用法是定义一个 Adapter，它允许外部类的实例被看作是另一个不相关的类的实例。

如下：

```java
private final class KeySet extends AbstractSet<K> {
        public Iterator<K> iterator() {
            return newKeyIterator();
        }
        public int size() {
            return size;
        }
        public boolean contains(Object o) {
            return containsKey(o);
        }
        public boolean remove(Object o) {
            return HashMap.this.removeEntryForKey(o) != null;
        }
        public void clear() {
            HashMap.this.clear();
        }
    }
```

如果声明成员类不要求访问外围实例，就要声明成 static，使他成为静态成员类。会节省内存和时间。

匿名类的常见语法是动态地创建函数对象。

如果一个嵌套类需要在单个方法之外仍然是可见，或者它太长了，不适合于放在方法内部，就应该使用成员类。如果成员类的每个实例都需要一个指向其外围实例的引用，就要把成员类做成非静态的。否则，应该做成静态的。假设这个嵌套类属于一个方法的内部，如果你只需要在一个地方创建实例，并且已经有了一个预置的类型就可以说明这个类的特征，就要把它做成匿名类。

【参考资料】

1. [Effective Java](http://book.douban.com/subject/3360807/)

---EOF---

