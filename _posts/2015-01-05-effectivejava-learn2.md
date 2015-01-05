---
layout: post
title: effective-java对象通用方法
categories: effective-java
tags: 笔记
---

#### 覆盖 equals 方法

如果要重写 equals 方法，需要遵守下面几条规定：

- 自反性：对于任何非空引用值 x，x.equals(x) 都应返回 true。
- 对称性：对于任何非空引用值 x 和 y，当且仅当 y.equals(x) 返回 true 时，x.equals(y) 才应返回 true。
- 传递性：对于任何非空引用值 x、y 和 z，如果 x.equals(y) 返回 true，并且 y.equals(z) 返回 true，那么 x.equals(z) 应返回 true。
- 一致性：对于任何非空引用值 x 和 y，多次调用 x.equals(y) 始终返回 true 或始终返回 false，前提是对象上 equals 比较中所用的信息没有被修改。
- 对于任何非空引用值 x，x.equals(null) 都应返回 false。

下面是编写高质量 equals 方法的诀窍：

1. 使用 == 操作检查引用是否相同。
2. 使用 instanceof 检查类型是否正确。
3. 把参数转换成正确的类型。
4. 对于该类中的每个关键字段进行匹配是否相同。

如下代码省略了 get 和 set 方法，eclipse 自动生成的 equals 方法：

```java
public class City
{
    private String zipCode;

    private String name;

    @Override
    public boolean equals(Object obj)
    {
        if (this == obj)
            return true;
        if (obj == null)
            return false;
        if (getClass() != obj.getClass())
            return false;
        City other = (City) obj;
        if (name == null)
        {
            if (other.name != null)
                return false;
        }
        else if (!name.equals(other.name))
            return false;
        if (zipCode == null)
        {
            if (other.zipCode != null)
                return false;
        }
        else if (!zipCode.equals(other.zipCode))
            return false;
        return true;
    }
}
```

可以使用 guava 或者 jdk7 提供的工具类 Objects 来简化 equals 方法：

```java
public class City
{
    private String zipCode;

    private String name;

    @Override
    public boolean equals(Object obj)
    {
        if (this == obj)
            return true;
        if (obj == null)
            return false;
        if (getClass() == obj.getClass())
        {
            City that = (City) obj;
            return Objects.equals(name, that.name) && Objects.equals(zipCode, that.zipCode);
        }
        return false;
    }
}
```

equals 方法的代码如下：

```java
public static boolean equals(Object a, Object b) {
        return (a == b) || (a != null && a.equals(b));
    }
```

#### 覆盖 equals 时总要覆盖 hashCode

在每个覆盖了 equals 方法的类中，也必须覆盖 hashCode 方法。下面是 Object 对象的规范：

- 在应用程序的执行期间，只要对象的 equals 方法的比较操作所用到的信息没有被修改，那么对这同一个对象多次调用 hashCode 方法都必须返回同一个整数。从某一应用程序的一次执行到同一应用程序的另一次执行，该整数无需保持一致。
- 如果两个对象根据 equals 方法比较是相等的，那么调用这两个对象的 hashCode 方法产生的结果必须相同。
- 如果两个对象根据 equals 方法比较不相等，那么调用两个对象的 hashCode 方法，则不一定产生不同的整数结果。

如果不覆盖 hashCode 方法，那么该类在集合 HashMap、HashSet 和 Hashtable 种使用会出现问题。

可以看 HashMap 当中的代码：

```java
final int hash(Object k) {
        int h = hashSeed;
        if (0 != h && k instanceof String) {
            return sun.misc.Hashing.stringHash32((String) k);
        }

        h ^= k.hashCode();

        // This function ensures that hashCodes that differ only by
        // constant multiples at each bit position have a bounded
        // number of collisions (approximately 8 at default load factor).
        h ^= (h >>> 20) ^ (h >>> 12);
        return h ^ (h >>> 7) ^ (h >>> 4);
    }
```

在 HashMap 当中 put 的时候，会先计算对象的 hash 值来确定对象在哈西表的位置。

#### 始终覆盖 toString

虽然 Object 类提供类 toString 方法的缩写，如下：

```java
    public String toString() {
        return getClass().getName() + "@" + Integer.toHexString(hashCode());
    }
```

但是这个 toString 表达的信息不是很丰富，所以我们一般会自己实现 toString 方法。

#### 谨慎的覆盖 clone

Object 类种有个 clone 方法，但是如果你的类不实现 Cloneable 接口就调用 clone 方法会抛出 CloneNotSupportedException 异常。

创建和返回该对象的一个拷贝。一般的含义是，对于任何对象 x，表达式：

x.clone() != x 将会是 true，并且 x.clone().getClass() == x.getClass() 也是 true。

通常 x.clone().equals(x) 也会是 true。

如下代码：

```java
public class Person implements Cloneable {
     private String name;
     private int age;
     private String sex;
     @Override
     protected Person clone() throws CloneNotSupportedException {
          return (Person) super.clone();
     }
}
     public static void main(String[] args) throws Exception {
          Person person = new Person();
          Person person2 = person.clone();

          System.out.println(person);
          System.out.println(person2);
     }
```

输出结果如下：

```java
test.Person@b412c18
test.Person@63b5e16d
```

#### 考虑实现 Comparable 接口

类实现了 Comparable 接口就代表它的实例具有内在的排序关系。给实现 Comparable 接口的对象数组进行排序如下：

```java
Arrays.sort(a);
```

Comparable 接口的定义如下:

```java
public interface Comparable<T> {
    public int compareTo(T o);
}
```

比较整数基本类型的域，可以使用关系操作符 `<` 、 `>`。浮点数用 Double.compare 或者 Float.compare 。

【参考资料】

1. [Effective Java](http://book.douban.com/subject/3360807/)

---EOF---

