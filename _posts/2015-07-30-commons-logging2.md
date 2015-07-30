---
layout: post
title: commons-logging 源码分析 2
categories: log
tags: java, log
---

## 日志级别

 commons-logging 当中的 log 接口定义了一下几个日志级别：

- fatal 非常严重的错误，导致系统中止。期望这类信息能立即显示在状态控制台上。
- error 其它运行期错误或不是预期的条件。期望这类信息能立即显示在状态控制台上。
- warn 使用了不赞成使用的 API 、非常拙劣使用 API, ' 几乎就是 ' 错误 , 其它运行时不合需要和不合预期的状态但还没必要称为 " 错误 " 。期望这类信息能立即显示在状态控制台上。
- info 运行时产生的有意义的事件。期望这类信息能立即显示在状态控制台上。
- debug 系统流程中的细节信息。期望这类信息仅被写入 log 文件中。
- trace 更加细节的信息。期望这类信息仅被写入 log 文件中。

## Log4JLogger 实现

在 commons-logging 只是相当于 jdbc 接口的作用真是日志的调用都是其他的实现。简单看下 Log4j 的日志实现适配。

```java
public Log4JLogger(String name) {
        this.name = name;
        this.logger = getLogger();
    }

public Logger getLogger() {
        Logger result = logger;
        if (result == null) {
            synchronized(this) {
                result = logger;
                if (result == null) {
                    logger = result = Logger.getLogger(name);// Log4j 获取日志的方式
                }
            }
        }
        return result;
    }

public void info(Object message, Throwable t) {
        getLogger().log(FQCN, Level.INFO, message, t);
    }
```

其他的适配 Log 实现基本类似。

## 缺点

common-logging 通过动态查找的机制，在程序运行时自动找出真正使用的日志库。由于它使用了 ClassLoader 寻找和载入底层的日志库， 导致了象 OSGI 这样的框架无法正常工作，因为 OSGI 的不同的插件使用自己的 ClassLoader。 OSGI 的这种机制保证了插件互相独立，然而却使 Apache Common-Logging 无法工作。

## 优点

提供了接口形式的日志框架，使我们不比依赖具体的日志框架，耦合度比较小。

---EOF---
