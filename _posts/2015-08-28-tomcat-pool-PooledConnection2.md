---
layout: post
title: tomcat 数据库连接池之 ConnectionPool 下
categories: 数据库连接池
tags: java, 连接池
---

上篇我们分析了，连接池应该有的基本功能，那么这次我们来分析下连接池的一些其他的功能。

- 定时清理无响应的连接
- 异步获取连接
- 对真实 Connection 的包装和增强

### 定时清理连接

定时清理器会做如下工作：

- 清理 busy 队列当中的泄露的连接，即大于 removeAbandonedTimeout 的连接
- 如果当前 idle 队列的大小大于 minIdle，会清除 idle 队列的连接

在初始化连接池的时候会初始化清理任务：

```java
public void initializePoolCleaner(PoolConfiguration properties) {
        //if the evictor thread is supposed to run, start it now
        if (properties.isPoolSweeperEnabled()) {
            poolCleaner = new PoolCleaner(this, properties.getTimeBetweenEvictionRunsMillis());
            poolCleaner.start();
        } //end if
    }

protected static class PoolCleaner extends TimerTask {
        protected WeakReference<ConnectionPool> pool;// 弱引用连接池
        protected long sleepTime;// timeBetweenEvictionRunsMillis 多久进行一次清理
        protected volatile long lastRun = 0;

        PoolCleaner(ConnectionPool pool, long sleepTime) {
            this.pool = new WeakReference<>(pool);
            this.sleepTime = sleepTime;
            if (sleepTime <= 0) {
                log.warn("Database connection pool evicter thread interval is set to 0, defaulting to 30 seconds");
                this.sleepTime = 1000 * 30;
            } else if (sleepTime < 1000) {
                log.warn("Database connection pool evicter thread interval is set to lower than 1 second.");
            }
        }

        @Override
        public void run() {
            ConnectionPool pool = this.pool.get();
            if (pool == null) {
                stopRunning();
            } else if (!pool.isClosed() &&
                    (System.currentTimeMillis() - lastRun) > sleepTime) {
                lastRun = System.currentTimeMillis();
                try {
                    if (pool.getPoolProperties().isRemoveAbandoned())
                        pool.checkAbandoned();// 移除 busy 队列当中泄露的连接
                    if (pool.getPoolProperties().getMinIdle() < pool.idle
                            .size())
                        pool.checkIdle();// 移除 idle 队列当中超过数量的连接
                    if (pool.getPoolProperties().isTestWhileIdle())//默认值是 false
                        pool.testAllIdle();// 移除不是空闲连接状态的连接
                } catch (Exception x) {
                    log.error("", x);
                }
            }
        }

        public void start() {
            registerCleaner(this);
        }

        public void stopRunning() {
            unregisterCleaner(this);
        }
    }
```

### 异步获取连接

看如下代码：

```java
public Future<Connection> getConnectionAsync() throws SQLException {
        try {
            PooledConnection pc = borrowConnection(0, null, null);// 传入 0，代表不进行等待
            if (pc!=null) {
                return new ConnectionFuture(pc);
            }
        }catch (SQLException x) {
            if (x.getMessage().indexOf("NoWait")<0) {
                throw x;
            }
        }
        if (idle instanceof FairBlockingQueue<?>) {
            Future<PooledConnection> pcf = ((FairBlockingQueue<PooledConnection>)idle).pollAsync();
            return new ConnectionFuture(pcf);
        } else if (idle instanceof MultiLockFairBlockingQueue<?>) {
                Future<PooledConnection> pcf = ((MultiLockFairBlockingQueue<PooledConnection>)idle).pollAsync();
                return new ConnectionFuture(pcf);
        } else {
            throw new SQLException("Connection pool is misconfigured,
            								doesn't support async retrieval. Set the 'fair' property to 'true'");
        }
    }
```

首先还是调用 borrowConnection 方法，传入的等待时间是 0，如果获取到了连接，使用 ConnectionFuture 包装下返回，如果没获取，通过 idle 队列的 pollAsync 方法获取连接，然后在通过 ConnectionFuture 包装后返回。

### 对真实 Connection 的包装和增强

再来看下 setupConnection 这个方法，在上篇已经简单的看过这个方法了。

```java
protected Connection setupConnection(PooledConnection con) throws SQLException {
        JdbcInterceptor handler = con.getHandler();// PooledConnection 内部有拦截器，会有一些增强的功能
        if (handler==null) {
            handler = new ProxyConnection(this,con,getPoolProperties().isUseEquals());// 创建默认的 handler
            PoolProperties.InterceptorDefinition[] proxies = getPoolProperties().getJdbcInterceptorsAsArray();
	        // 这个循环是把 ProxyConnection 放在链的最后，如果 JdbcInterceptor 的顺序是 A，B
            // 那么这个链就是 A -> B -> ProxyConnection
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

首先了解下思路：

- 先从 PooledConnection 当中获取 getHandler
- 如果获取到了 handler,根据这个 handler 来创建 jdbc Connection 的动态代理类

在来看看 JdbcInterceptor 的源码：

```java
// 实现了 InvocationHandler 原来创建动态代理类
public abstract class JdbcInterceptor implements InvocationHandler {

    public static final String CLOSE_VAL = "close";// jdbc Connection 当中的一些方法名字定义成了常量

    public static final String TOSTRING_VAL = "toString";

    public static final String ISCLOSED_VAL = "isClosed";

    public static final String GETCONNECTION_VAL = "getConnection";

    public static final String UNWRAP_VAL = "unwrap";

	public static final String ISWRAPPERFOR_VAL = "isWrapperFor";

    public static final String ISVALID_VAL = "isValid";

    public static final String EQUALS_VAL = "equals";

    public static final String HASHCODE_VAL = "hashCode";

    protected Map<String,InterceptorProperty> properties = null;

    private volatile JdbcInterceptor next = null;

    private boolean useEquals = true;

    public JdbcInterceptor() {
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    	// 链式调用拦截器
        if (getNext()!=null) return getNext().invoke(proxy,method,args);
        else throw new NullPointerException();
    }

    public JdbcInterceptor getNext() {
        return next;
    }

	public void setNext(JdbcInterceptor next) {
        this.next = next;
    }

    public boolean compare(String name1, String name2) {
        if (isUseEquals()) {
            return name1.equals(name2);
        } else {
            return name1==name2;
        }
    }

    public boolean compare(String methodName, Method method) {
        return compare(methodName, method.getName());
    }

    public abstract void reset(ConnectionPool parent, PooledConnection con);

    public void disconnected(ConnectionPool parent, PooledConnection con, boolean finalizing) {
    }

    public Map<String,InterceptorProperty> getProperties() {
        return properties;
    }

    public void setProperties(Map<String,InterceptorProperty> properties) {
        this.properties = properties;
        final String useEquals = "useEquals";
        InterceptorProperty p = properties.get(useEquals);
        if (p!=null) {
            setUseEquals(Boolean.parseBoolean(p.getValue()));
        }
    }

    public boolean isUseEquals() {
        return useEquals;
    }

    public void setUseEquals(boolean useEquals) {
        this.useEquals = useEquals;
    }

    public void poolClosed(ConnectionPool pool) {
        // NOOP
    }

    public void poolStarted(ConnectionPool pool) {
        // NOOP
    }

}
```

JdbcInterceptor 是个抽象类，主要的 invoke 方法就是链式调用下一个 JdbcInterceptor。

然后在来看看 ProxyConnection 的实现，继承了 JdbcInterceptor。

```java
public class ProxyConnection extends JdbcInterceptor {

    protected PooledConnection connection = null;

    protected ConnectionPool pool = null;

    public PooledConnection getConnection() {
        return connection;
    }

    public void setConnection(PooledConnection connection) {
        this.connection = connection;
    }

    public ConnectionPool getPool() {
        return pool;
    }

    public void setPool(ConnectionPool pool) {
        this.pool = pool;
    }

    protected ProxyConnection(ConnectionPool parent, PooledConnection con,
            boolean useEquals) {
        pool = parent;
        connection = con;
        setUseEquals(useEquals);
    }

    @Override
    public void reset(ConnectionPool parent, PooledConnection con) {
        this.pool = parent;
        this.connection = con;
    }

    public boolean isWrapperFor(Class<?> iface) {
        if (iface == XAConnection.class && connection.getXAConnection()!=null) {
            return true;
        } else {
            return (iface.isInstance(connection.getConnection()));
        }
    }


    public Object unwrap(Class<?> iface) throws SQLException {
        if (iface == PooledConnection.class) {
            return connection;
        }else if (iface == XAConnection.class) {
            return connection.getXAConnection();
        } else if (isWrapperFor(iface)) {
            return connection.getConnection();
        } else {
            throw new SQLException("Not a wrapper of "+iface.getName());
        }
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        if (compare(ISCLOSED_VAL,method)) {
            return Boolean.valueOf(isClosed());
        }
        if (compare(CLOSE_VAL,method)) {// 如果是 close 操作，那么释放连接到 idle 队列
            if (connection==null) return null; //noop for already closed.
            PooledConnection poolc = this.connection;
            this.connection = null;
            pool.returnConnection(poolc);
            return null;
        } else if (compare(TOSTRING_VAL,method)) {
            return this.toString();
        } else if (compare(GETCONNECTION_VAL,method) && connection!=null) {
            return connection.getConnection();
        } else if (method.getDeclaringClass().equals(XAConnection.class)) {
            try {
                return method.invoke(connection.getXAConnection(),args);
            }catch (Throwable t) {
                if (t instanceof InvocationTargetException) {
                    throw t.getCause() != null ? t.getCause() : t;
                } else {
                    throw t;
                }
            }
        }
        if (isClosed()) throw new SQLException("Connection has already been closed.");
        if (compare(UNWRAP_VAL,method)) {
            return unwrap((Class<?>)args[0]);
        } else if (compare(ISWRAPPERFOR_VAL,method)) {
            return Boolean.valueOf(this.isWrapperFor((Class<?>)args[0]));
        }
        try {
            PooledConnection poolc = connection;
            if (poolc!=null) {
                return method.invoke(poolc.getConnection(),args);
            } else {
                throw new SQLException("Connection has already been closed.");
            }
        }catch (Throwable t) {
            if (t instanceof InvocationTargetException) {
                throw t.getCause() != null ? t.getCause() : t;
            } else {
                throw t;
            }
        }
    }

    public boolean isClosed() {
        return connection==null || connection.isDiscarded();
    }

    public PooledConnection getDelegateConnection() {
        return connection;
    }

    public ConnectionPool getParentPool() {
        return pool;
    }

    @Override
    public String toString() {
        return "ProxyConnection["+(connection!=null?connection.toString():"null")+"]";
    }

}
```

有很多内置的 JdbcInterceptor 来实现各种扩展功能，不多介绍。

---EOF---
