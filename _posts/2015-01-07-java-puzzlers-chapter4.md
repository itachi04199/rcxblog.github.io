---
layout: post
title: Java 解惑 -- 异常谜题
categories: Java解惑
tags: 笔记
---

#### 异常返回

看下面这段代码：

```java
public class Indecisive {
    public static void main(String[] args) {
        System.out.println(decision());
    }
    static boolean decision() {
        try {
            return true;
        } finally {
            return false;
        }
    }
}
```

上面的代码输出结果是 false。

详细的 try-catch-finally 的详见[这里](http://renchx.com/java-exception/)

#### 是否编译报错

```java
public class Arcane1 {
    public static void main(String[] args) {
        try {
            System.out.println("Hello world");
        } catch(IOException e) {
            System.out.println("I've never seen
               println fail!");
        }
    }
}
```

上面的例子在编译的时候是报错的，如果一个 catch 子句要捕获一个类型为 E 的被检查异常，而其相对应的 try 子句不能抛出 E 的某种子类型的异常，那么这就是一个编译期错误。

```java
public class Arcane2 {
    public static void main(String[] args) {
        try {
            // If you have nothing nice to say, say nothing
        } catch(Exception e) {
            System.out.println("This can't  happen");
        }
    }
}
```

它之所以可以编译，是因为它唯一的 catch 子句检查了 Exception。尽管 JLS 在这一点上十分含混不清，但是捕获 Exception 或 Throwble 的 catch 子句是合法的，不管与其相对应的 try 子句的内容为何。

看下面例子：

```java
interface Type1 {
    void f() throws CloneNotSupportedException;
}

interface Type2 {
    void f() throws InterruptedException;
}

interface Type3 extends Type1, Type2 {
}

public class Arcane3 implements Type3 {
    public void f() {
        System.out.println("Hello world");
    }
    public static void main(String[] args) {
        Type3 t3 = new Arcane3();
        t3.f();
    }
}
```

这个程序可以正常编译，Type1 和 Type2 都限制了方法抛出的异常，Type3 的 f 方法要抛出的被检查异常集合的交集，而不是合集。
因此，静态类型为 Type3 的对象上的f方法根本就不能抛出任何被检查异常。

#### 流异常

看下面例子：

```java
static void copy(String src, String dest) throws IOException
    {
        InputStream in = null;
        OutputStream out = null;
        try
        {
            in = new FileInputStream(src);
            out = new FileOutputStream(dest);
            byte[] buf = new byte[1024];
            int n;
            while ((n = in.read(buf)) > 0)
                out.write(buf, 0, n);
        }
        finally
        {
            if (in != null)
                in.close();
            if (out != null)
                out.close();
        }
    }
```

问题在 finally 语句块自身中。close 方法也可能会抛出 IOException异常。如果这正好发生在 in.close 被调用之时，那么这个异常就会阻止 out.close 被调用，从而使输出流仍保持在开放状态。

可以修改成如下：

```java
finally {
     if (in != null) {
          try {
              in.close();
          } catch (IOException ex) {
              // There is nothing we can do if close fails
          }
     if (out != null)
          try {
              out.close();
          } catch (IOException ex) {
              // There is nothing we can do if close fails
          }
    }
}
```

在 jdk1.5 之后可以如下修改：

```java
finally {
     closeIgnoringException(in);
     closeIgnoringEcception(out);
}
private static void closeIgnoringException(Closeable c) {
     if (c != null) {
           try {
             c.close();
           } catch (IOException ex) {
             // There is nothing we can do if close  fails
           }
     }
}
```

【参考资料】

1.  [java解惑](http://book.douban.com/subject/1473329/)

---EOF---

