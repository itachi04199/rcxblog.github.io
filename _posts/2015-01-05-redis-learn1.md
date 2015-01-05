---
layout: post
title: redis字符串学习
categories: redis
tags: redis
---

#### 入门热身

获取符合规则的键名列表：

```
KEYS pattern
```

pattern 支持 glob 风格通配符：

符号|含义
:--|:--
?|匹配一个字符
*|匹配任意个字符
[]|匹配括号间的任意一个字符
\x|匹配字符 x，用于转义符号。如果要匹配 ? 就需要使用 \?

SET 命令创建 key：

```
127.0.0.1:6379> SET name rcx
OK
```

使用 KEYS * 就可以获取到 Redis 当中所有的 key 了。

```
127.0.0.1:6379> KEYS *
1) "name"
```

判断一个键是否存在：如果键存在则返回整数类型 1， 否则返回 0。

```
127.0.0.1:6379> EXISTS name
(integer) 1
127.0.0.1:6379> EXISTS rcx
(integer) 0
```

删除键：可以删除一个或多个键，返回值是删除的键的个数。

```
127.0.0.1:6379> DEL name ww
(integer) 2
127.0.0.1:6379> DEL name ww
(integer) 0
```

第二次执行 DEL 命令因为键已经被删除，实际上没有删除任何键，所以返回 0。

获取键值的数据类型:

```
127.0.0.1:6379> SET name rcx
OK
127.0.0.1:6379> TYPE name
string
```

#### 字符串类型

字符串是 Redis 中最基本的数据类型，它能存储任何形式的字符串，包括二进制数据。

递增数字：当存储的字符串是整型时，Redis 提供了一个实用的命令 INCR，其作用是让当前键值递增，并返回递增后的值。

```
127.0.0.1:6379> INCR num
(integer) 1
127.0.0.1:6379> INCR num
(integer) 2
127.0.0.1:6379> type num
string
```

当要操作的键不存在时会没人键值是 0，所以第一次递增后的结果是 1。

当键值不是整数时 Redis 会提示错误：

```
127.0.0.1:6379> INCR name
(error) ERR value is not an integer or out of range
```

Redis 对于键的命名没有强制的要求，但比较好的实践是用 “对象类型:对象ID:对象属性” 来命名一个键，如 user:1:friends 来存储 ID 为 1 的用户的好友列表。

增加指定的整数：INCRBY 命令与 INCR 命令基本一样，只是可以通过参数指定一次增加的数值

```
127.0.0.1:6379> set num 2
OK
127.0.0.1:6379> INCRBY num 2
(integer) 4
```

减少指定的整数：与递增用法相同

```
127.0.0.1:6379> decr num
(integer) 3
127.0.0.1:6379> DECRBY num 2
(integer) 1
```

增加浮点数：INCRBYFLOAT 命令与 INCRBY 命令类似，只是可以递增一个双精度浮点数

```
127.0.0.1:6379> set f 2.7
OK
127.0.0.1:6379> INCRBYFLOAT f 1
"3.7"
127.0.0.1:6379> INCRBYFLOAT f 5e+2
"503.70000000000000001"
```

向尾部追加值：返回值是追加后字符串的总长度。

```
127.0.0.1:6379> set name rcx
OK
127.0.0.1:6379> APPEND name asd
(integer) 6
127.0.0.1:6379> GET name
"rcxasd"
```

获取字符串长度：如果该键不存在则返回 0。

```
127.0.0.1:6379> STRLEN name
(integer) 6
```

同时获得/设置多个键值：

```
127.0.0.1:6379> MSET a 1 b 2 c 3
OK
127.0.0.1:6379> MGET a b c
1) "1"
2) "2"
3) "3"
```

【参考资料】

1. [Redis入门指南](http://book.douban.com/subject/24522045/)

---EOF---
