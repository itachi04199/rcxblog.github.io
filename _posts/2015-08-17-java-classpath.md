---
layout: post
title: 理解 java classpath
categories: java基础
tags: java
---

classpath 是一个命令行参数，或环境变量，它告诉 Java 虚拟机或者 Java 编译器到哪里寻找用户定义的类和包。

## 概述

类似于经典的动态加载的行为，执行 Java 程序时，Java 虚拟机发现并进行懒加载类。classpath 告诉 Java 在哪里查找文件系统中定义这些类的文件。

虚拟机按如下顺序进行查找：

1. bootstrap classes ：这些类是 JAVA 平台的基础类，例如 java.lang.Object
2. extension classes ：包是在 JRE 或 JDK，jre/lib/ext 目录下
3. 用户自定义类和类库

默认情况下的 JDK 的标准 API 和扩展包，无需设置在哪里可以找到他们。所有用户定义的包和库的路径必须在命令行中进行设置。

## 例子

在 org.mypackage 包含如下类：

- HelloWorld (main class)
- SupportClass
- UtilClass

并且这些文件的物理目录是 D:\myprogram (on Windows) or /home/user/myprogram (on Linux).

然后目录结构如下：

```java
Microsoft Windows
D:\myprogram\
      |
      ---> org\
            |
            ---> mypackage\
                     |
                     ---> HelloWorld.class
                     ---> SupportClass.class
                     ---> UtilClass.class

Linux
/home/user/myprogram/
            |
            ---> org/
                  |
                  ---> mypackage/
                           |
                           ---> HelloWorld.class
                           ---> SupportClass.class
                           ---> UtilClass.class
```

当我们要执行程序的时候需要进行如下输入在命令行当中：

```bash
Microsoft Windows
 java -classpath D:\myprogram org.mypackage.HelloWorld
Linux
 java -cp /home/user/myprogram org.mypackage.HelloWorld
```

## 设置 classpath

### 通过环境变量形式设置 classpath

```java
set CLASSPATH=D:\myprogram
java org.mypackage.HelloWorld
```

使用 -classpath 选项可以覆盖 CLASSPATH 环境变量，如果不声明，则当前目录作为 classpath。这就意味着如果我们当前目录是 `D:\myprogram\` , 我们就不需要显式地指定类路径。

### 设置一个 jar 文件的路径

如果程序使用 supportLib.jar ，物理上的目录 D:\myprogram\lib\ 相应的物理文件的结构是：

```bash
D:\myprogram\
      |
      ---> lib\
            |
            ---> supportLib.jar
      |
      ---> org\
            |
            --> mypackage\
                       |
                       ---> HelloWorld.class
                       ---> SupportClass.class
                       ---> UtilClass.class
```

然后命令行执行如下：

```bash
java -classpath D:\myprogram;D:\myprogram\lib\supportLib.jar org.mypackage.HelloWorld
```

或者

```bash
set CLASSPATH=D:\myprogram;D:\myprogram\lib\supportLib.jar
java org.mypackage.HelloWorld
```

添加目录下的所有 jar：

```bash
java -classpath ".;c:\mylib\*" MyApp
```

## 编写 Manifest 文件

假设我们有一个 helloWorld.jar 文件，在 `D:\myprogram ` 目录下，我们有如下的文件结构：

```bash
D:\myprogram\
      |
      ---> helloWorld.jar
      |
      ---> lib\
            |
            ---> supportLib.jar
```

Manifest 文件可以如下定义：

```java
Main-Class: org.mypackage.HelloWorld
Class-Path: lib/supportLib.jar
```

可以如下进行执行 jar :

```java
java -jar D:\myprogram\helloWorld.jar [app arguments]
```

多个 jar 的引用可以如下设置：

```bash
Class-Path: lib/supportLib.jar lib/supportLib2.jar
```

【参考资料】

1. https://en.wikipedia.org/wiki/Classpath_(Java)

---EOF---
