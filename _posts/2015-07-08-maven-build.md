---
layout: post
title: maven 属性以及灵活构建
categories: maven
tags: java, maven
---

## maven 属性

一种简单的 maven 属性的使用是：

```xml
<properties>
	<org.springframework.version>3.1.4.RELEASE</org.springframework.version>
</properties>
```

上面这种属性是 maven 的自定义属性。

maven 有六种属性：

- 内置属性：主要两个内置属性，${basedir}表示项目根目录，即包含 pom.xml 文件的目录；${version} 标识项目版本。
- pom 属性：可以使用该类型的属性引用 pom 文件总对应元素的值。${project.artifacId} 就对应了 artifactId 元素的值。总是以 project 开头，下面是一些常用的 pom 属性：
    - ${project.build.sourceDirectory}：项目主源码目录，默认是 src/main/java
    - ${project.build.testSourceDirectory}:项目测试源码目录。
    - ${project.build.directory}:项目构建输出目录
- 自定义属性：上面的例子就是自定义属性。
- setting 属性：跟 pom 属性同理，使用 setting 开头，引用 setting.xml 文件中 xml 元素的值。
- java 系统属性：所有的 java 系统属性 maven 都可以引用，可以使用 mvn help:system 来查看所有 java 系统属性。
- 环境变量属性：所有的环境变量都可以使用 env. 开头的 maven 属性引用。例如： env.JAVA_HOME 代表 JAVA_HOME 环境变量的值。可以使用 mvn help:system 来查看所有 java 系统属性。

看下面第一个例子：

```xml
<dependency>
  <groupId>${project.groupId}</groupId>
  <artifactId>account-email</artifactId>
  <version>${project.version}</version>
</dependency>
<dependency>
  <groupId>${project.groupId}</groupId>
  <artifactId>account-persist</artifactId>
  <version>${project.version}</version>
</dependency>
```

这个例子就是当前的模块依赖 account-email 和 account-persist，这三个模块使用相同的 groupId 和 version。这样当项目版本升级的时候，就不需要更改依赖的版本了。

## 构建环境差异

在项目 src/main/resources 目录下我们会放入 jdbc.properties 属性文件，在开发和测试和线上环境我们需要连接的是不同的数据库环境。

我们的想法是可以把不同环境的配置文件放入到不同的文件夹当中。

所以可以们可以在 src/main 目录下面创建文件夹，大致如下:

```
src/main
	- profiles
    	- dev
        - beta
        - prod
```

maven 为了提供不同环境的 profile 有可以配置的节点：

```xml
<profiles>
<profile>
  <id>dev</id>
  <properties>
    <profile.dir>dev</profile.dir>
  </properties>
</profile>

<profile>
  <id>beta</id>
  <properties>
    <profile.dir>beta</profile.dir>
  </properties>
</profile>

<profile>
  <id>prod</id>
  <properties>
    <profile.dir>prod</profile.dir>
  </properties>
</profile>
</profiles>
```

当我们要激活不同环境的 profile 的时候可以使用如下命令：

```bash
mvn clean install -Pdev
```

使用 -P 参数加上 profile 的 id 来激活 profile。同样还有很多的激活方式，命令方式激活比较清楚。

同时，如果我们要想在 build 的时候加载不同的配置文件需要在 pom 当中的 build 节点下，添加对资源的处理：

```xml
<resources>
  <resource>
    <directory>src/main/profiles/.${profile.dir}</directory>
  </resource>
  <resource>
    <directory>src/main/resources</directory>
  </resource>
</resources>
```

当然上面对资源的处理使用 maven-resources-plugin 插件。

这样我们就实现了灵活构建不同环境的配置文件。

【参考资料】

1. [maven 实战](http://book.douban.com/subject/5345682/)

---EOF--
