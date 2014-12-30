#### 考虑用静态工厂方法代替构造器

静态工厂方法与构造器不同的第一大优势在于，它们有名称。

如果构造方法的参数过于太多，那么当我们创建对象的时候就必须弄清楚每个参数的意义。如果使用静态工厂方法来创建我们可以明确的知道这个方法会返回什么样子的对象。

静态工厂方法与构造器不同的第二大优势在于，不必在每次调用它们的时候都创建一个新对象。

大家都知道的就是单例模式，每次调用都返回相同的对象。

静态工厂方法与构造器不同的第三大优势在于，它们可以返回原返回类型的任何子类型的对象。

在 JDK 当中，Collections.unmodifiableList 方法会返回一个不可变的 List，但是我们不需要知道这个不可变类的信息，对于使用 api 的用户来说，减轻很大的负担。

当然每次调用静态方法返回的类型也可能根据参数不同返回的类型不一样。例如，EnumSet 类没有构造函数，它的返回就是通过参数不一样返回的类型不一样，如果它的元素小于等于 64 个，这个静态方法会返回 RegularEnumSet 类型，否则返回 JumboEnumSet 类型。

静态工厂方法与构造器不同的第四大优势在于，在创建参数化类型实例的时候，它们使代码变得更加简洁。

如下代码：

```java
Map<String, List<String>> map = new HashMap<String, List<String>>();
```

上面这个创建 map 的例子当中，我们不得不重复写 2 次类型参数。类型参数越复杂这个声明语句就越复杂。但是使用静态方法，编译器会帮助我们找到类型参数，这是类型推导。

```java
public static <K, V> HashMap<K, V> newHashMap() {
	return new HashMap<K, V>();
}
```

静态工厂方法的缺点是，类如果不含有公有的或者受保护的构造器，就不能被子类化。第二个缺点就是它们与其他的静态方法实际上没有任何区别。

下面是静态工厂方法的一些惯用名称：

方法名|含义
:--|:--
valueOf|该方法返回的实例与它的参数有相同的值。String.valueOf()
of|valueOf 的简写，在 EnumSet 中有。
getInstance|返回的实例是通过方法的参数来描述的，不一定与参数有相同的值。对于单例来说，该方法没参数，并且返回唯一的实例。
newInstance|每次可以返回不同的实例。
getType|跟 getInstance 类似。Type 表示工厂方法所返回的对象类型。
newType|跟 newInstance 类似。Type 表示工厂方法所返回的对象类型。

#### 遇到多个构造器参数时要考虑用构建器

看如下代码：

```java
package test;

public class Num {
	private int field1;
	private int field2;
	private int field3;
	private int field4;
	private int field5;

	public int getField1() {
		return field1;
	}

	public Num(int field1) {
		this.field1 = field1;
	}

	public Num(int field1, int field2) {
		this.field1 = field1;
		this.field2 = field2;
	}

	public Num(int field1, int field2, int field3) {
		this.field1 = field1;
		this.field2 = field2;
		this.field3 = field3;
	}

	public void setField1(int field1) {
		this.field1 = field1;
	}

	public int getField2() {
		return field2;
	}

	public void setField2(int field2) {
		this.field2 = field2;
	}

	public int getField3() {
		return field3;
	}

	public void setField3(int field3) {
		this.field3 = field3;
	}

	public int getField4() {
		return field4;
	}

	public void setField4(int field4) {
		this.field4 = field4;
	}

	public int getField5() {
		return field5;
	}

	public void setField5(int field5) {
		this.field5 = field5;
	}
}
```

上面这个类只是举例说明，如果构造方法参数过多，并且重载的也多。那么当你创建对象的时候就必须仔细分析每一个参数是上面含义。

如果不通过多参数的构造方法来创建对象也可以使用空构造方法来创建对象，然后使用 setter 方法来设置每个必要的参数。但是使用这种方式会导致没构造完成对象之前使用对象的错误，即线程不安全。

当然最好的方式就是使用 Builder 模式，先获得个构造器对象，然后根据构造器 build 出相应的对象。

```java
package test;

public class Num {
	private final int field1;
	private final int field2;
	private final int field3;
	private final int field4;
	private final int field5;

	public Num(Build build) {
		this.field1 = build.field1;
		this.field2 = build.field2;
		this.field3 = build.field3;
		this.field4 = build.field4;
		this.field5 = build.field5;
	}

	static class Build {
		private int field1;
		private int field2;
		private int field3;
		private int field4;
		private int field5;

		public static Build newBuilder() {
			return new Build();
		}

		public Build addFiled1(int field) {
			this.field1 = field;
			return this;
		}

		public Build addFiled2(int field) {
			this.field2 = field;
			return this;
		}

		public Build addFiled3(int field) {
			this.field3 = field;
			return this;
		}

		public Build addFiled4(int field) {
			this.field4 = field;
			return this;
		}

		public Build addFiled5(int field) {
			this.field5 = field;
			return this;
		}

		public Num build() {
			return new Num(this);
		}
	}

	public static void main(String[] args) {
		Num num = Build.newBuilder().addFiled1(1).addFiled2(2).addFiled3(3)
				.addFiled4(4).addFiled5(5).build();

	}
}
```

使用构造器模式很容易编写，更重要的是易于阅读。构造器模式的方便在于，你可以随意添加几个参数，不像构造器那样死板。

当然构造器模式也有缺点，那就是在创建对象前需要先创建构造器对象，这个创建构造器的开销不容小觑。并且，构造器模式代码很冗长，因此它只适合在参数很多情况下使用。

#### 避免创建不必要的对象

一般来说能重用对象尽量重用对象，这样就避免创建多余的对象。

如下：

```java
String str = new String("str");
```

什么这样的方式每一次都会重新创建一个新的对象。可以使用如下方式:

```java
String str = "str";
```

还有在代码当中注意不必要的装箱和拆箱：

```java
     public static void main(String[] args) {
        long a = System.currentTimeMillis();
        Long sum = 0L;
        for (int i = 1; i < Integer.MAX_VALUE; i++)
        {
            sum = sum + i;
        }
        System.out.println(sum);
        System.out.println(System.currentTimeMillis() - a);
    }
```

上面这段代码使用 Long 类型的，这意味着大概创建了 2 的 31 次幂个多余的 Long 实例。程序大概执行了 7 秒，如果改成基本类型的 long 程序执行不到 1 秒。

#### 消除过期的对象引用

看下面的这个类：

```java
public class Stack
{
    private Object[] elements;
    private int size;
    private static final int DEFAULT_INIT_CAPACITY = 16;

    public Stack()
    {
        this(DEFAULT_INIT_CAPACITY);
    }

    public Stack(int capacity)
    {
        elements = new Object[capacity];
    }

    public void push(Object element)
    {
        ensureCapacity();
        elements[size++] = element;
    }

    public Object pop()
    {
        if (size == 0)
        {
            throw new RuntimeException("stack is empty");
        }
        return elements[--size];
    }

    private void ensureCapacity()
    {
        if (size == elements.length)
        {
            elements = Arrays.copyOf(elements, size * 2);
        }
    }
}
```

这个类在使用的时候有可能发生内存泄漏。如果栈是先增长，然后再收缩，那么从栈中弹出来的元素不会被当作垃圾回收。这是因为，栈内部维护着对这些对象的过期引用。

过期引用就是指永远也不会再被解除的引用。如果一个对象被无意识的保留起来了，那么垃圾回收不会处理这个对象，并且也不会处理被这个对象所引用的所有其他对象。

这个例子可以做如下修改：

```java
public Object pop()
    {
        if (size == 0)
        {
            throw new RuntimeException("stack is empty");
        }
        Object element = elements[--size];
        elements[size] = null;
        return element;
    }
```

清空对象引用应该是一种例外，而不是一种规范行为。消除过期引用最好的方法就是让包含该引用的变量结束其生命周期。

【参考资料】

1. [Effective Java](http://book.douban.com/subject/3360807/)

---EOF---
