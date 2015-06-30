---
layout: post
title: maven 5分钟入门
categories: maven
tags: java, maven
---

安装好 maven 后，在命令行执行如下命令：

```bash
mvn -version
```

如果安装成功后会出现如下返回：

```bash
Apache Maven 3.3.1 (cab6659f9874fa96462afef40fcf6bc033d58c1c; 2015-03-14T04:10:27+08:00)
Maven home: /usr/local/Cellar/maven/3.3.1/libexec
Java version: 1.7.0_71, vendor: Oracle Corporation
Java home: /Library/Java/JavaVirtualMachines/jdk1.7.0_71.jdk/Contents/Home/jre
Default locale: zh_CN, platform encoding: UTF-8
OS name: "mac os x", version: "10.10.3", arch: "x86_64", family: "mac"
```

## 创建项目

如果我们需要创建一个简单的项目使用如下：

```bash
mvn archetype:generate -DgroupId=com.mycompany.app -DartifactId=my-app -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
```

第一次执行的时候回比较慢，需要下载一些插件到本地。

生成的项目结构如下：

```bash
my-app
|-- pom.xml
`-- src
    |-- main
    |   `-- java
    |       `-- com
    |           `-- mycompany
    |               `-- app
    |                   `-- App.java
    `-- test
        `-- java
            `-- com
                `-- mycompany
                    `-- app
                        `-- AppTest.java
```

`src/main/java` 里面是放项目代码的，`src/test/java` 里面是放测试代码的。

在文件夹根目录有 pom.xml 文件，是 Project Object Model。

## PMO

生成的 pom.xm 文件如下：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
 
  <groupId>com.mycompany.app</groupId>
  <artifactId>my-app</artifactId>
  <version>1.0-SNAPSHOT</version>
  <packaging>jar</packaging>
 
  <name>Maven Quick Start Archetype</name>
  <url>http://maven.apache.org</url>
 
  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.8.2</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
</project>
```

当我们执行 maven 的目标 archetype:generate，并且传入一些参数给这个目标。前缀 archetype 是包含目标的插件。我们基于这个插件创建一个简单的项目。

## 构建项目

```bash
mvn package
```

回输出如下：

```bash
...
[INFO] Building jar: /Users/renchunxiao/Documents/maventest/my-app/target/my-app-1.0-SNAPSHOT.jar
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 3.580 s
[INFO] Finished at: 2015-06-30T11:06:50+08:00
[INFO] Final Memory: 14M/245M
[INFO] ------------------------------------------------------------------------
```

这次我们执行的时候不像 archetype:generate 我们只需要输入一个单词 package。与目标不同，这次是阶段，一个阶段是构建生命周期当中的一步。

执行我们刚才生成的 jar

```bash
>java -cp my-app-1.0-SNAPSHOT.jar com.mycompany.app.App
Hello World!
```

## 运行 maven 工具

maven 阶段，尽管几乎没有一个全面的列表，这些都是执行最常见的默认生命周期阶段。

- validae：检查项目
- compile：编译项目代码
- test：执行单元测试
- package：代码打包
- integration-test：集成测试
- verify：运行任何检查，验证包是有效的，并符合质量标准
- install：安装包到本地资源库，供其他项目使用
- deploy：部署到线上环境

另外两个 Maven 的生命周期超过上面的默认列表:

- clean：清除构建过的项目
- site：生成项目文档

【参考资料】

1. [maven 官网5分钟入门](http://maven.apache.org/guides/getting-started/maven-in-five-minutes.html)

---EOF---

