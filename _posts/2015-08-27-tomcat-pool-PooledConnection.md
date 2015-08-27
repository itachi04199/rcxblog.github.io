---
layout: post
title: tomcat 数据库连接池之 ConnectionPool 上
categories: 数据库连接池
tags: java, 连接池
---

## 数据库连接池

> 数据库连接是一种关键的有限的昂贵的资源，这一点在多用户的网页应用程序中体现得尤为突出。对数据库连接的管理能显著影响到整个应用程序的伸缩性和健壮性，影响到程序的性能指标。数据库连接池正是针对这个问题提出来的。数据库连接池负责分配、管理和释放数据库连接，它允许应用程序重复使用一个现有的数据库连接，而不是再重新建立一个；释放空闲时间超过最大空闲时间的数据库连接来避免因为没有释放数据库连接而引起的数据库连接遗漏。这项技术能明显提高对数据库操作的性能。

## Tomcat jdbc pool

Tomcat 在 7.0 以前的版本都是使用 commons-dbcp 做为连接池的实现，但是目前都切换到 Tomcat jdbc pool 了。具体的看下面的介绍：

http://tomcat.apache.org/tomcat-8.0-doc/jdbc-pool.html

### ConnectionPool

首先我们来分析下，最基本的连接池会提供什么样的功能，并且以什么样的数据结构出现。

连接池首先应该提供的主要功能：

- 获取连接
- 释放连接：真实关闭连接
- 返回连接：将使用完的连接放回到池

连接池的数据结构：

- 连接容器：存放提前创建的连接

上面是一个连接池框架应该提供的最基本的功能，那么看一下 tomcat jdbc pool 是如何实现的。首先是看下 ConnectionPool 内部的数据结构：

```java
// 使用阻塞队列来存放已经在使用的连接
private BlockingQueue<PooledConnection> busy;

// 使用阻塞队列来存放空闲的连接
private BlockingQueue<PooledConnection> idle;
```

具体的执行是，我们从 idle 当中获取连接，如果有那么把这个连接放到 busy 当中。当返回连接的时候是一个反过来的流程。

当 idle 当中没有空闲连接的时候，会去判断，如果没达到最大连接数就去创建新的连接，如果到达了就等待，超过等待时间就超时。

### 初始化 ConnectionPool

再来看下 ConnectionPool 的初始化过程：

```java
public ConnectionPool(PoolConfiguration prop) throws SQLException {
        //setup quick access variables and pools
        init(prop);
    }

protected void init(PoolConfiguration properties) throws SQLException {
        poolProperties = properties;

        //...省略代码

        busy = new LinkedBlockingQueue<>();
        if (properties.isFairQueue()) // 如果是公平的创建的阻塞队列不一样
            idle = new FairBlockingQueue<>();
        } else {
            idle = new LinkedBlockingQueue<>();
        }

        initializePoolCleaner(properties);

         //...省略代码

        PooledConnection[] initialPool = new PooledConnection[poolProperties.getInitialSize()];
        try {
            for (int i = 0; i < initialPool.length; i++) {//循环创建连接
                initialPool[i] = this.borrowConnection(0, null, null);
            }

        } catch (SQLException x) {
            log.error("Unable to create initial connections of pool.", x);
            //...省略代码
        } finally {
            for (int i = 0; i < initialPool.length; i++) {
                if (initialPool[i] != null) {
                    try {this.returnConnection(initialPool[i]);}catch(Exception x){/*NOOP*/}
                }
            }
        }
        closed = false;
    }
```

上面 init 方法省略了部分代码，主要就是 borrowConnection 和 returnConnection 两个方法。

先循环创建 N 个连接，并且放到了 busy 的队列中，然后在 finally 当中归还连接，这样就初始化了 N 个闲置的连接了。

- borrowConnection：借连接，如果没连接的话会去创建一个，还会有一些其他的判断
- returnConnection：返还连接，里面也会判断是否需要真实关闭的情况

再来看 borrowConnection 代码的实现：

```java
//wait 参数，等待连接的时间，如果传入的是0，那么不等待。如果传入的是-1，那么使用配置的 maxWait。
private PooledConnection borrowConnection(int wait, String username, String password) throws SQLException {

        if (isClosed()) {
            throw new SQLException("Connection pool closed.");
        }

        long now = System.currentTimeMillis();
        PooledConnection con = idle.poll();//先从空闲队列当中获取一个

        while (true) {
            if (con!=null) {//空闲队列当中有空闲连接
                PooledConnection result = borrowConnection(now, con, username, password);
                //null should never be returned, but was in a previous impl.
                if (result!=null) return result;
            }

            if (size.get() < getPoolProperties().getMaxActive()) {// 判断 size 是不是小于最大的活跃连接数
                if (size.addAndGet(1) > getPoolProperties().getMaxActive()) {// 如果自增完大于最大活跃数，则不能创建连接
                    size.decrementAndGet();
                } else {
                    return createConnection(now, con, username, password);//创建新连接
                }
            }

            long maxWait = wait;
            if (wait==-1) {
                maxWait = (getPoolProperties().getMaxWait()<=0)?Long.MAX_VALUE:getPoolProperties().getMaxWait();
            }

            long timetowait = Math.max(0, maxWait - (System.currentTimeMillis() - now));
            waitcount.incrementAndGet();
            try {
                con = idle.poll(timetowait, TimeUnit.MILLISECONDS);//通过传递超时时间来获取连接
            } catch (InterruptedException ex) {
                if (getPoolProperties().getPropagateInterruptState()) {
                    Thread.currentThread().interrupt();
                }
                SQLException sx = new SQLException("Pool wait interrupted.");
                sx.initCause(ex);
                throw sx;
            } finally {
                waitcount.decrementAndGet();
            }
            if (maxWait==0 && con == null) {
                if (jmxPool!=null) {
                    jmxPool.notify(org.apache.tomcat.jdbc.pool.jmx.ConnectionPool.POOL_EMPTY, "Pool empty - no wait.");
                }
                throw new PoolExhaustedException("[" + Thread.currentThread().getName()+"] " +
                        "NoWait: Pool empty. Unable to fetch a connection, none available["+busy.size()+" in use].");
            }

			if (con == null) {//超时没获取到连接
                if ((System.currentTimeMillis() - now) >= maxWait) {
                    if (jmxPool!=null) {
                        jmxPool.notify(org.apache.tomcat.jdbc.pool.jmx.ConnectionPool.POOL_EMPTY, "Pool empty - timeout.");
                    }
                    throw new PoolExhaustedException("[" + Thread.currentThread().getName()+"] " +
                        "Timeout: Pool empty. Unable to fetch a connection in " + (maxWait / 1000) +
                        " seconds, none available[size:"+size.get() +"; busy:"+busy.size()+"; idle:"+idle.size()+"; lastwait:"+timetowait+"].");
                } else {
                    //no timeout, lets try again
                    continue;
                }
            }
        } //while
    }
```

上面代码看着很长，其实逻辑很简单：

- 先从空闲队列当中获取连接
	- 如果获取到了：返回
    - 没获取到连接：判断是不是需要新创建连接。
    	- 如果不能新创建：设置超时时间，如果获取到返回，没获取到抛出异常
        - 可以创建：创建新连接，并且返回

当然上面的代码当中还有几个问题我们没弄明白：

- createConnection 会如何创建真实的新连接
- 当从 idle 队列当中获取到了连接，那么 borrowConnection(now, con, username, password); 会做什么？

那么一一分析， createConnection 源代码：

```java
protected PooledConnection createConnection(long now, PooledConnection notUsed, String username, String password) throws SQLException {
        PooledConnection con = create(false);
        if (username!=null) con.getAttributes().put(PooledConnection.PROP_USER, username);
        if (password!=null) con.getAttributes().put(PooledConnection.PROP_PASSWORD, password);
        boolean error = false;
        try {
            con.lock();
            con.connect();
            if (con.validate(PooledConnection.VALIDATE_INIT)) {
                con.setTimestamp(now);
                if (getPoolProperties().isLogAbandoned()) {
                    con.setStackTrace(getThreadDump());
                }
                if (!busy.offer(con)) {//新创建的连接会加入到 busy 队列当中
                    log.debug("Connection doesn't fit into busy array, connection will not be traceable.");
                }
                return con;
            } else {
                throw new SQLException("Validation Query Failed, enable logValidationErrors for more details.");
            }
        } catch (Exception e) {
            error = true;
            // .. 省略
        } finally {
            if (error ) {
                release(con);
            }
            con.unlock();
        }
    }

protected PooledConnection create(boolean incrementCounter) {
        if (incrementCounter) size.incrementAndGet();
        PooledConnection con = new PooledConnection(getPoolProperties(), this);
        return con;
    }
```

PooledConnection 是对 jdbc 原生的 Connection 的包装，具体以后分析。

在分析 borrowConnection 的重载方法：

```java
protected PooledConnection borrowConnection(long now, PooledConnection con, String username, String password) throws SQLException {
        // 进入了这个方法代表我们从 idle 队列当中获取到了连接，那么需要对这个连接进行一些设置
        boolean setToNull = false;
        try {
            con.lock();
            if (con.isReleased()) {
                return null;
            }

            //是否需要强制重新连接
            boolean forceReconnect = con.shouldForceReconnect(username, password) || con.isMaxAgeExpired();

            if (!con.isDiscarded() && !con.isInitialized()) {
            	// 连接没被废弃，但是真实的 connection 是 null
                forceReconnect = true;
            }

            if (!forceReconnect) {//不需要重新连接
                if ((!con.isDiscarded()) && con.validate(PooledConnection.VALIDATE_BORROW)) {
                    con.setTimestamp(now);
                    if (getPoolProperties().isLogAbandoned()) {
                        con.setStackTrace(getThreadDump());
                    }
                    if (!busy.offer(con)) {
                        log.debug("Connection doesn't fit into busy array, connection will not be traceable.");
                    }
                    return con;
                }
            }

			try {
                con.reconnect();
                int validationMode = getPoolProperties().isTestOnConnect() || getPoolProperties().getInitSQL()!=null ?
                    PooledConnection.VALIDATE_INIT :
                    PooledConnection.VALIDATE_BORROW;

                if (con.validate(validationMode)) {
                    con.setTimestamp(now);
                    if (getPoolProperties().isLogAbandoned()) {
                        con.setStackTrace(getThreadDump());
                    }
                    if (!busy.offer(con)) {
                        log.debug("Connection doesn't fit into busy array, connection will not be traceable.");
                    }
                    return con;
                } else {
                    release(con);
                    setToNull = true;
                    throw new SQLException("Failed to validate a newly established connection.");
                }
            } catch (Exception x) {
                release(con);
                setToNull = true;
                if (x instanceof SQLException) {
                    throw (SQLException)x;
                } else {
                    SQLException ex  = new SQLException(x.getMessage());
                    ex.initCause(x);
                    throw ex;
                }
            }
        } finally {
            con.unlock();
            if (setToNull) {
                con = null;
            }
        }
    }
```

最后分析下 returnConnection 返还连接的逻辑：

```java
protected void returnConnection(PooledConnection con) {
        if (isClosed()) {
            release(con);
            return;
        }

        if (con != null) {
            try {
                con.lock();

                if (busy.remove(con)) {

                    if (!shouldClose(con,PooledConnection.VALIDATE_RETURN)) {//判断当前连接是否需要真实关闭
                        con.setStackTrace(null);
                        con.setTimestamp(System.currentTimeMillis());
                        if (((idle.size()>=poolProperties.getMaxIdle()) && !poolProperties.isPoolSweeperEnabled()) ||
                         (!idle.offer(con))) {// 判断空闲队列大于最大空闲数量，或者添加到空闲队列失败，会释放这个连接
                            if (log.isDebugEnabled()) {
                                log.debug("Connection ["+con+"] will be closed and not returned to the pool,
                                idle["+idle.size()+"]>=maxIdle["+poolProperties.getMaxIdle()+"] idle.offer failed.");
                            }
                            release(con);
                        }
                    } else {
                        if (log.isDebugEnabled()) {
                            log.debug("Connection ["+con+"] will be closed and not returned to the pool.");
                        }
                        release(con);
                    } //end if
                } else {
                    if (log.isDebugEnabled()) {
                        log.debug("Connection ["+con+"] will be closed and not returned to the pool, busy.remove failed.");
                    }
                    release(con);
                }
            } finally {
                con.unlock();
            }
        } //end if
    }
```

### 获取连接

如下源代码：

```java
public Connection getConnection() throws SQLException {
        //check out a connection
        PooledConnection con = borrowConnection(-1,null,null);
        return setupConnection(con);
    }

protected Connection setupConnection(PooledConnection con) throws SQLException {
        //每个连接都有一个代理，获取以前缓存的代理
        JdbcInterceptor handler = con.getHandler();
        if (handler==null) {
            handler = new ProxyConnection(this,con,getPoolProperties().isUseEquals());
            PoolProperties.InterceptorDefinition[] proxies = getPoolProperties().getJdbcInterceptorsAsArray();
            for (int i=proxies.length-1; i>=0; i--) {
                try {
                    JdbcInterceptor interceptor = proxies[i].getInterceptorClass().newInstance();
                    interceptor.setProperties(proxies[i].getProperties());
                    interceptor.setNext(handler);
                    interceptor.reset(this, con);
                    handler = interceptor;
                }catch(Exception x) {
                    SQLException sx = new SQLException("Unable to instantiate interceptor chain.");
                    sx.initCause(x);
                    throw sx;
                }
            }
            con.setHandler(handler);
        } else {
            JdbcInterceptor next = handler;
            while (next!=null) {
                next.reset(this, con);
                next = next.getNext();
            }
        }

        try {
            getProxyConstructor(con.getXAConnection() != null);
            Connection connection = null;
            //动态代理，来实现对 PooledConnection 的包装
            if (getPoolProperties().getUseDisposableConnectionFacade() ) {
                connection = (Connection)proxyClassConstructor.newInstance(new Object[] { new DisposableConnectionFacade(handler) });
            } else {
                connection = (Connection)proxyClassConstructor.newInstance(new Object[] {handler});
            }
            return connection;
        }catch (Exception x) {
            SQLException s = new SQLException();
            s.initCause(x);
            throw s;
        }
    }
```

先在 idle 队列当中获取一个空闲的连接，然后包装这个连接。

首先，为什么需要对 PooledConnection 进行包装？因为，我们返回去的连接当这个使用完之后需要关闭的时候，我们不能真正的关闭，需要回收到 idle 队列当中，tomcat jdbc pool 使用了 JDK 的动态代理机制来实现。

本篇结束，下篇继续分析。

---EOF---
