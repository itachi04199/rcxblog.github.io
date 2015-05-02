---
layout: post
title: Java中ArrayList源码分析
categories: java基础
tags: 笔记
---

>List 接口的大小可变数组的实现。实现了所有可选列表操作，并允许包括 null 在内的所有元素。除了实现 List 接口外，此类还提供一些方法来操作内部用来存储列表的数组的大小。（此类大致上等同于 Vector 类，除了此类是不同步的。）

>size、isEmpty、get、set、iterator 和 listIterator 操作都以固定时间运行。add 操作以分摊的固定时间 运行，也就是说，添加 n 个元素需要 O(n) 时间。其他所有操作都以线性时间运行（大体上讲）。与用于 LinkedList 实现的常数因子相比，此实现的常数因子较低。

>每个 ArrayList 实例都有一个容量。该容量是指用来存储列表元素的数组的大小。它总是至少等于列表的大小。随着向 ArrayList 中不断添加元素，其容量也自动增长。并未指定增长策略的细节，因为这不只是添加元素会带来分摊固定时间开销那样简单。

ArrayList 的头信息如下：

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
```

ArrayList的基本使用方式：

```java
    List<String> list = new ArrayList<String>();
    list.add("string1");
    list.add("string2");
    list.set(1, "ss2");
    System.out.println(list.get(1));
    System.out.println(list.indexOf("string1"));
    System.out.println(list.size());
```

看下ArrayList当中定义的变量：

```java
    private static final int DEFAULT_CAPACITY = 10;//列表默认容量

    private static final Object[] EMPTY_ELEMENTDATA = {};

    private transient Object[] elementData;//添加到list当中的元素都存放在这个数组当中

    private int size;
```

下面是ArrayList的无参构造方法(jdk1.7)：

```java
	public ArrayList() {
        super();
        this.elementData = EMPTY_ELEMENTDATA;
    }
```

可以看出将空数组赋值给了elementData；下面在看下当第一次调用add()方法会如何处理：

```java
	public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }

    private void ensureCapacityInternal(int minCapacity) {
        if (elementData == EMPTY_ELEMENTDATA) {
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }

        ensureExplicitCapacity(minCapacity);
    }

    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }

    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
```

可以看出来当第一次进行add操作的时候，调用ensureCapacityInternal(1)方法。

由于默认构造方法的时候elementData是等于EMPTY_ELEMENTDATA的所以数组的最小容量minCapacity会被赋值成默认的大小为10。

紧接着调用ensureExplicitCapacity(10)方法。在ensureExplicitCapacity方法中minCapacity的值是传入进来的10，elementData元素是空数组所以长度是0，会进去grow(10)方法当中。

grow方法会给内部的elementData数组进行扩容。

看下ArrayList带参数的构造方法：

```java
	public ArrayList(int initialCapacity) {
        super();
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        this.elementData = new Object[initialCapacity];
    }

    public ArrayList(Collection<? extends E> c) {
        elementData = c.toArray();
        size = elementData.length;
        // c.toArray might (incorrectly) not return Object[] (see 6260652)
        if (elementData.getClass() != Object[].class)
            elementData = Arrays.copyOf(elementData, size, Object[].class);
    }
```

下面来分析remove方法：

```java
	public E remove(int index) {
        rangeCheck(index);

        modCount++;
        E oldValue = elementData(index);

        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work

        return oldValue;
    }

    private void rangeCheck(int index) {
        if (index >= size)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }

    E elementData(int index) {
        return (E) elementData[index];
    }
```

首先判断传入进来的索引是否大于size，如果大于则抛出异常。

然后获取elementData数组中的元素。求出这个元素后面还有几个元素numMoved，如果numMoved大于0进行数组移动。

再来看另外重载的add方法：

```java
	public void add(int index, E element) {
        rangeCheckForAdd(index);

        ensureCapacityInternal(size + 1);  // Increments modCount!!
        System.arraycopy(elementData, index, elementData, index + 1,
                         size - index);
        elementData[index] = element;
        size++;
    }
    private void rangeCheckForAdd(int index) {
        if (index > size || index < 0)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }
    private void ensureCapacityInternal(int minCapacity) {
        if (elementData == EMPTY_ELEMENTDATA) {
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }

        ensureExplicitCapacity(minCapacity);
    }
    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }
    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
```

在指定索引的位置添加元素，处理过程如下：

首先，判断索引是否合法。
然后判断内部数组elementData是否需要扩容，如果需要则扩容。
最后，将索引处开始元素向后移动，然后将新元素添加到索引位置当中。

再来看set方法，set方法是将指定索引的位置元素用新元素进行替换，并且返回老元素：

```java
	public E set(int index, E element) {
        rangeCheck(index);

        E oldValue = elementData(index);
        elementData[index] = element;
        return oldValue;
    }

    private void rangeCheck(int index) {
        if (index >= size)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }
```

看下remove方法的另外重载方法：

```java
	public boolean remove(Object o) {
        if (o == null) {
            for (int index = 0; index < size; index++)
                if (elementData[index] == null) {
                    fastRemove(index);
                    return true;
                }
        } else {
            for (int index = 0; index < size; index++)
                if (o.equals(elementData[index])) {
                    fastRemove(index);
                    return true;
                }
        }
        return false;
    }
    private void fastRemove(int index) {
        modCount++;
        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work
    }
```

这个是移除传入的元素，首先判断传入的元素是否为null。如果为null则对元素的比较使用==，如果不为null则使用对象的equals方法比较。fastRemove的处理跟remove(int index)里面的处理逻辑一样。

最后看看定位元素索引的方法，indexOf和lastIndexOf方法：

```java
	public int indexOf(Object o) {
        if (o == null) {
            for (int i = 0; i < size; i++)
                if (elementData[i]==null)
                    return i;
        } else {
            for (int i = 0; i < size; i++)
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }
    public int lastIndexOf(Object o) {
        if (o == null) {
            for (int i = size-1; i >= 0; i--)
                if (elementData[i]==null)
                    return i;
        } else {
            for (int i = size-1; i >= 0; i--)
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }
```

lastIndexOf就从数组后面往前面遍历，思路很开阔。

以上就是ArrayList大致的实现方式和一些方法的源代码。总体上来看是很简单的容器，里面的实现方式也很值得我们去学习。

---EOF---

