---
layout: post
title: commons-logging 源码分析 1
categories: log
tags: java, log
---

## 介绍

有关 java 当中的日志介绍，这里不多说，给几个链接：

http://blog.csdn.net/yycdaizi/article/details/8276265

## 框架分析

![包结构](http://renchx.com/public/images/commons-logging.png)

使用方式：

```java
	private static Log log = LogFactory.getLog(CommonsLog.class);

	public static void main(String[] args) {
		log.info("rcx");
	}

```

主要看 LogFactory 类：

```java
public static Log getLog(Class clazz) throws LogConfigurationException {
        return getFactory().getInstance(clazz);
    }


public static LogFactory getFactory() throws LogConfigurationException {
        ClassLoader contextClassLoader = getContextClassLoaderInternal();

        if (contextClassLoader == null) {
            if (isDiagnosticsEnabled()) {
                logDiagnostic("Context classloader is null.");
            }
        }

        // 如果我们不是第一次使用 LogFactory 我们会从当前缓存里面获取 LogFactory 的实例
        LogFactory factory = getCachedFactory(contextClassLoader);
        if (factory != null) {
            return factory;
        }

        if (isDiagnosticsEnabled()) {
            logDiagnostic(
                    "[LOOKUP] LogFactory implementation requested for the first time for context classloader " +
                    objectId(contextClassLoader));
            logHierarchy("[LOOKUP] ", contextClassLoader);
        }

        // 获取 commons-logging.properties 里面的配置
        Properties props = getConfigurationFile(contextClassLoader, FACTORY_PROPERTIES);

        ClassLoader baseClassLoader = contextClassLoader;
        if (props != null) {
            String useTCCLStr = props.getProperty(TCCL_KEY);
            if (useTCCLStr != null) {
                if (Boolean.valueOf(useTCCLStr).booleanValue() == false) {
                    baseClassLoader = thisClassLoader;
                }
            }
        }

        if (isDiagnosticsEnabled()) {
            logDiagnostic("[LOOKUP] Looking for system property [" + FACTORY_PROPERTY +
                          "] to define the LogFactory subclass to use...");
        }

        try {
            String factoryClass = getSystemProperty(FACTORY_PROPERTY, null);
            if (factoryClass != null) {
                if (isDiagnosticsEnabled()) {
                    logDiagnostic("[LOOKUP] Creating an instance of LogFactory class '" + factoryClass +
                                  "' as specified by system property " + FACTORY_PROPERTY);
                }
                factory = newFactory(factoryClass, baseClassLoader, contextClassLoader);
            } else {
                if (isDiagnosticsEnabled()) {
                    logDiagnostic("[LOOKUP] No system property [" + FACTORY_PROPERTY + "] defined.");
                }
            }
        } catch (SecurityException e) {
            if (isDiagnosticsEnabled()) {
                logDiagnostic("[LOOKUP] A security exception occurred while trying to create an" +
                              " instance of the custom factory class" + ": [" + trim(e.getMessage()) +
                              "]. Trying alternative implementations...");
            }
        } catch (RuntimeException e) {
            if (isDiagnosticsEnabled()) {
                logDiagnostic("[LOOKUP] An exception occurred while trying to create an" +
                              " instance of the custom factory class" + ": [" +
                              trim(e.getMessage()) +
                              "] as specified by a system property.");
            }
            throw e;
        }

        if (factory == null) {
            if (isDiagnosticsEnabled()) {
                logDiagnostic("[LOOKUP] Looking for a resource file of name [" + SERVICE_ID +
                              "] to define the LogFactory subclass to use...");
            }
            try {
                final InputStream is = getResourceAsStream(contextClassLoader, SERVICE_ID);

                if( is != null ) {
                    BufferedReader rd;
                    try {
                        rd = new BufferedReader(new InputStreamReader(is, "UTF-8"));
                    } catch (java.io.UnsupportedEncodingException e) {
                        rd = new BufferedReader(new InputStreamReader(is));
                    }

                    String factoryClassName = rd.readLine();
                    rd.close();

                    if (factoryClassName != null && ! "".equals(factoryClassName)) {
                        if (isDiagnosticsEnabled()) {
                            logDiagnostic("[LOOKUP]  Creating an instance of LogFactory class " +
                                          factoryClassName +
                                          " as specified by file '" + SERVICE_ID +
                                          "' which was present in the path of the context classloader.");
                        }
                        factory = newFactory(factoryClassName, baseClassLoader, contextClassLoader );
                    }
                } else {
                    if (isDiagnosticsEnabled()) {
                        logDiagnostic("[LOOKUP] No resource file with name '" + SERVICE_ID + "' found.");
                    }
                }
            } catch (Exception ex) {
                if (isDiagnosticsEnabled()) {
                    logDiagnostic(
                        "[LOOKUP] A security exception occurred while trying to create an" +
                        " instance of the custom factory class" +
                        ": [" + trim(ex.getMessage()) +
                        "]. Trying alternative implementations...");
                }
            }
        }


        if (factory == null) {
            if (props != null) {
                if (isDiagnosticsEnabled()) {
                    logDiagnostic(
                        "[LOOKUP] Looking in properties file for entry with key '" + FACTORY_PROPERTY +
                        "' to define the LogFactory subclass to use...");
                }
                String factoryClass = props.getProperty(FACTORY_PROPERTY);
                if (factoryClass != null) {
                    if (isDiagnosticsEnabled()) {
                        logDiagnostic(
                            "[LOOKUP] Properties file specifies LogFactory subclass '" + factoryClass + "'");
                    }
                    factory = newFactory(factoryClass, baseClassLoader, contextClassLoader);

                } else {
                    if (isDiagnosticsEnabled()) {
                        logDiagnostic("[LOOKUP] Properties file has no entry specifying LogFactory subclass.");
                    }
                }
            } else {
                if (isDiagnosticsEnabled()) {
                    logDiagnostic("[LOOKUP] No properties file available to determine" + " LogFactory subclass from..");
                }
            }
        }


        if (factory == null) {
            if (isDiagnosticsEnabled()) {
                logDiagnostic(
                    "[LOOKUP] Loading the default LogFactory implementation '" + FACTORY_DEFAULT +
                    "' via the same classloader that loaded this LogFactory" +
                    " class (ie not looking in the context classloader).");
            }

            // 创建 LogFactoryImpl 实例
            factory = newFactory(FACTORY_DEFAULT, thisClassLoader, contextClassLoader);
        }

        if (factory != null) {
            // 缓存这个 LogFactory
            cacheFactory(contextClassLoader, factory);
			// 把 commons-logging.properties 里面的属性放到 LogFactory 的属性里面
            if (props != null) {
                Enumeration names = props.propertyNames();
                while (names.hasMoreElements()) {
                    String name = (String) names.nextElement();
                    String value = props.getProperty(name);
                    factory.setAttribute(name, value);
                }
            }
        }

        return factory;
    }
```

从上面代码可以看到我们如果没有一些配置的话会获取到一个默认的 LogFactoryImpl 实例。

将上面的 getFactory 方法整理成下面几个步骤

1. 获得当前线程的 classloader，命名为 contextClassLoader
2. 根据 contextClassLoader 在缓存中获得 logFactory，这里的缓存使用的是org.apache.commons.logging.impl.WeakHashTable，key 是 classloader，value 是logFactory，如果在缓存中有 logFactory 则直接返回，没有进入下面的流程
3. 读取配置文件 commons-logging.properties
4. 如果读到了配置文件，判断其中 use_tccl 属性，然后设定 baseClassLoader 是使用 contextClassLoader 还是使用加载 本Cla ss文件的那 个classloader（名字为thisClassLoader）
5. 下面生成 logFactory，这里会有使用四种方式，依次来尝试生成。
		1. 通过系统属性中寻找org.apache.commons.logging.LogFactory的value值，根据值为类名生成logFactory
		2. 通过资源META-INF/services/org.apache.commons.logging.LogFactory，获得的值为类名生成logFactory
		3. 通过刚才读取的配置文件commons-logging.properties，从中获取以org.apache.commons.logging.LogFactory为key的value值，根据值为类名生成logFactory
		4. 如果上面都不成功的话，会使用默认的实现类org.apache.commons.logging.impl.LogFactoryImpl来生成logFactory
6. 将生成的 logFactory 缓存起来
7. 返回 logFactory

分析：我们使用 Commons Logging 和 log4j 一起使用。

我们需要在 classpath 下添加 commons-logging.properties 文件，并且文件里面只需有一行设置 Log 的实现类，`org.apache.commons.logging.Log=org.apache.commons.logging.impl.Log4JLogger`。

那么我们看下为什么会使用这个类。

```java
//在 LogFactory 当中 getFactory 方法最后的操作，说明，如果我们添加了这个配置文件，会把属性通过 setAttribute 设置进去。
	if (props != null) {
                Enumeration names = props.propertyNames();
                while (names.hasMoreElements()) {
                    String name = (String) names.nextElement();
                    String value = props.getProperty(name);
                    factory.setAttribute(name, value);
                }
            }

// LogFactoryImpl 当中 setAttribute 的实现，attributes 是个 Hashtable
protected Hashtable attributes = new Hashtable();
public void setAttribute(String name, Object value) {
        if (logConstructor != null) {
            logDiagnostic("setAttribute: call too late; configuration already performed.");
        }

        if (value == null) {
            attributes.remove(name);
        } else {
            attributes.put(name, value);
        }

        if (name.equals(TCCL_KEY)) {
            useTCCL = value != null && Boolean.valueOf(value.toString()).booleanValue();
        }
    }

// 继续看 LogFactory 当中 getLog 后半部分是 getInstance
public static Log getLog(Class clazz) throws LogConfigurationException {
        return getFactory().getInstance(clazz);
    }

// LogFactoryImpl 的 getInstance 实现
public Log getInstance(String name) throws LogConfigurationException {
        Log instance = (Log) instances.get(name);
        if (instance == null) {
            instance = newInstance(name);
            instances.put(name, instance);
        }
        return instance;
    }

// LogFactoryImpl 的 newInstance
protected Log newInstance(String name) throws LogConfigurationException {
        Log instance;
        try {
            if (logConstructor == null) {
                instance = discoverLogImplementation(name);//获取具体 Log 实现
            }
            else {
                Object params[] = { name };
                instance = (Log) logConstructor.newInstance(params);
            }
          省略......
    }

// LogFactoryImpl 的 discoverLogImplementation
private Log discoverLogImplementation(String logCategory)
        throws LogConfigurationException {
        ......省略

        String specifiedLogClassName = findUserSpecifiedLogClassName();
 		......省略
        }


// LogFactoryImpl 的 findUserSpecifiedLogClassName,这个方法看着很长，其实很简单就是从 hashTable 里面要获取属性，如果指定了就回返回实现类的类全名。
private String findUserSpecifiedLogClassName() {
        if (isDiagnosticsEnabled()) {
            logDiagnostic("Trying to get log class from attribute '" + LOG_PROPERTY + "'");
        }
        String specifiedClass = (String) getAttribute(LOG_PROPERTY);

        if (specifiedClass == null) { // @deprecated
            if (isDiagnosticsEnabled()) {
                logDiagnostic("Trying to get log class from attribute '" +
                              LOG_PROPERTY_OLD + "'");
            }
            specifiedClass = (String) getAttribute(LOG_PROPERTY_OLD);
        }

        if (specifiedClass == null) {
            if (isDiagnosticsEnabled()) {
                logDiagnostic("Trying to get log class from system property '" +
                          LOG_PROPERTY + "'");
            }
            try {
                specifiedClass = getSystemProperty(LOG_PROPERTY, null);
            } catch (SecurityException e) {
                if (isDiagnosticsEnabled()) {
                    logDiagnostic("No access allowed to system property '" +
                        LOG_PROPERTY + "' - " + e.getMessage());
                }
            }
        }

        if (specifiedClass == null) { // @deprecated
            if (isDiagnosticsEnabled()) {
                logDiagnostic("Trying to get log class from system property '" +
                          LOG_PROPERTY_OLD + "'");
            }
            try {
                specifiedClass = getSystemProperty(LOG_PROPERTY_OLD, null);
            } catch (SecurityException e) {
                if (isDiagnosticsEnabled()) {
                    logDiagnostic("No access allowed to system property '" +
                        LOG_PROPERTY_OLD + "' - " + e.getMessage());
                }
            }
        }

        if (specifiedClass != null) {
            specifiedClass = specifiedClass.trim();
        }

        return specifiedClass;
    }
```

上面的过程可以总结成几部：

1. 判断 instances.get(name) 获取的 log 是不是为空，为空去 newInstance
2. newInstance 当中会判断 logConstructor 是不是空，为空 discoverLogImplementation
3. discoverLogImplementation 会查找是不是自己定义实现的 Log 全路径，如果有就根据这个 class 全路径创建 log，如果没有就按照顺序创建 log 实例，顺序如下：
		- org.apache.commons.logging.impl.Jdk14Logger
		- org.apache.commons.logging.impl.Jdk13LumberjackLogger
		- org.apache.commons.logging.impl.SimpleLog


下次会分析这些 Log 的实现方式，以及这个仅有10多个类组成的框架给我们什么样的启发，以及他的优缺点。


---EOF---
