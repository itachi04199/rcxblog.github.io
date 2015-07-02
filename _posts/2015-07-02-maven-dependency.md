---
layout: post
title: maven 依赖学习
categories: maven
tags: java, maven
---

## 依赖范围

依赖范围就是来控制 classpath(编译、测试、运行) 的关系，maven 的依赖范围有如下几种：

- compile：编译依赖范围。如果没指定，就会默认指定该范围。此依赖范围的 maven 依赖对编译、测试、运行三种 classpath 都有效。
- test：测试依赖范围。只对测试 classpath 有效。如，Junit。
- provided：已提供依赖范围。此依赖范围对编译和测试 classpath 有效。如， servlet-api，编译和测试的时候需要，但在项目运行的时候，容器会提供，就不需要重复的引入。
- runtime：运行时依赖范围。此依赖范围对测试和运行 classpath 有效。如，jdbc 驱动实现，代码编译只需要 jdk 提供的接口就好，测试和运行才需要实际的 jdbc 驱动。
- system：系统范围依赖。此依赖范围对编译和测试 classpath 有效。一般不使用。

如下图：

依赖范围|编译classpath有效|测试classpath有效|运行时classpath有效|例子
:--|:--|:--|:--|:--
compile|Y|Y|Y|spring-core
test|N|Y|N|Junit
provided|Y|Y|N|servlet-api
runtime|N|Y|Y|jdbc 驱动实现
system|Y|Y|N|本地的，maven仓库之外的类库文件

## 传递依赖

maven 的传递依赖会自动引入当前引入的依赖的依赖。

假设 A 依赖 B ， B 依赖 C ，我们称 A 对 B 是第一直接依赖， B 对 C 是第二直接依赖， A 对 C 是传递依赖。下表显示了三者的关系：

![maven传递依赖](http://renchx.com/public/images/maven1.png)

## 依赖调解

项目 A 有如下依赖：A -> B -> C -> X(1.0)、A -> D -> X(2.0)， X 是 A 的传递依赖，但是两条路径上都有 X ，所以 maven 会选择路径比较短的 X(2.0)。

如果路径长度相同的，pom 中依赖生命的顺序决定了谁会被解析使用，顺序靠前的那个依赖优胜。

## 可选依赖

有如下依赖关系：A -> B, B -> X,Y。根据传递依赖，如果这3个依赖的范围都是 compile，那么 X、Y 就是 A 的 compile 传递依赖。如果 X、Y 是可选依赖，依赖就不会被传递。

```xml
<dependency>
			<groupId>joda-time</groupId>
			<artifactId>joda-time</artifactId>
			<version>2.7</version>
            <optional>true</optional>
</dependency>
```

## maven 实践

可以使用`mvn dependency:list`命令查看 maven 已解析依赖：

```bash
mvn dependency:list

[INFO] The following files have been resolved:
[INFO]    org.apache.activemq:activemq-client:jar:5.11.1:compile
[INFO]    org.apache.geronimo.specs:geronimo-j2ee-management_1.1_spec:jar:1.0.1:compile
[INFO]    org.fusesource.hawtbuf:hawtbuf:jar:1.11:compile
[INFO]    org.hamcrest:hamcrest-core:jar:1.3:test
[INFO]    org.apache.geronimo.specs:geronimo-jms_1.1_spec:jar:1.1.1:compile
[INFO]    org.slf4j:slf4j-api:jar:1.7.10:compile
[INFO]    junit:junit:jar:4.12:test
[INFO]    javax.jms:jms:jar:1.1:compile
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
```

可以使用`mvn dependency:tree`命令查看当前项目的依赖树：

```bash
mvn dependency:tree

[INFO] --- maven-dependency-plugin:2.8:tree (default-cli) @ my-app ---
[INFO] com.mycompany.app:my-app:jar:1.0-SNAPSHOT
[INFO] +- javax.jms:jms:jar:1.1:compile
[INFO] +- junit:junit:jar:4.12:test
[INFO] |  \- org.hamcrest:hamcrest-core:jar:1.3:test
[INFO] \- org.apache.activemq:activemq-client:jar:5.11.1:compile
[INFO]    +- org.slf4j:slf4j-api:jar:1.7.10:compile
[INFO]    +- org.apache.geronimo.specs:geronimo-jms_1.1_spec:jar:1.1.1:compile
[INFO]    +- org.fusesource.hawtbuf:hawtbuf:jar:1.11:compile
[INFO]    \- org.apache.geronimo.specs:geronimo-j2ee-management_1.1_spec:jar:1.0.1:compile
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
```

可以使用`mvn dependency:analyze`可以详细的分析项目的依赖。

```bash
mvn dependency:analyze

[INFO] >>> maven-dependency-plugin:2.8:analyze (default-cli) > test-compile @ my-app >>>
[INFO]
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ my-app ---
[WARNING] Using platform encoding (UTF-8 actually) to copy filtered resources, i.e. build is platform dependent!
[INFO] skip non existing resourceDirectory /Users/renchunxiao/Documents/maventest/my-app/src/main/resources
[INFO]
[INFO] --- maven-compiler-plugin:3.1:compile (default-compile) @ my-app ---
[INFO] Nothing to compile - all classes are up to date
[INFO]
[INFO] --- maven-resources-plugin:2.6:testResources (default-testResources) @ my-app ---
[WARNING] Using platform encoding (UTF-8 actually) to copy filtered resources, i.e. build is platform dependent!
[INFO] skip non existing resourceDirectory /Users/renchunxiao/Documents/maventest/my-app/src/test/resources
[INFO]
[INFO] --- maven-compiler-plugin:3.1:testCompile (default-testCompile) @ my-app ---
[INFO] Nothing to compile - all classes are up to date
[INFO]
[INFO] <<< maven-dependency-plugin:2.8:analyze (default-cli) < test-compile @ my-app <<<
[INFO]
[INFO] --- maven-dependency-plugin:2.8:analyze (default-cli) @ my-app ---
[INFO] No dependency problems found
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 1.993 s
[INFO] Finished at: 2015-07-02T22:44:22+08:00
[INFO] Final Memory: 14M/310M
[INFO] ------------------------------------------------------------------------
```

【参考资料】

1. [maven 实战](http://book.douban.com/subject/5345682/)

---EOF---
