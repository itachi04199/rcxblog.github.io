---
layout: post
title: redis散列类型
categories: redis
tags: redis
---

#### 散列类型

散列类型不能嵌套其他的数据类型，其他的数据类型也不支持数据类型嵌套。

赋值和取值的命令：

```
HSET key field value
HGET key field
HMSET key field value [field value ...]
HMGET key field [filed ...]
HGETALL key
```

```
127.0.0.1:6379> HSET person name rcx
(integer) 1
127.0.0.1:6379> HSET person age 18
(integer) 1
127.0.0.1:6379> HSET person name rcx1
(integer) 0
127.0.0.1:6379> HGET person name
"rcx"
127.0.0.1:6379> HGETALL person
1) "name"
2) "rcx"
3) "age"
4) "18"
127.0.0.1:6379> TYPE person
hash
```

HSET 命令不区分插入和更新操作，当执行插入操作返回值是 1，当执行更新操作返回值是 0。

在 Redis 当中每个键都属于一个明确的数据类型，通过 HSET 命令建立的键是散列类型，通过 SET 命令建立的键是字符串类型。

HGETALL 返回的结果是字段和字段值组成的列表。

判断字段是否存在：如果存在返回 1，不存在返回 0。

```
127.0.0.1:6379> HEXISTS person name
(integer) 1
127.0.0.1:6379> HEXISTS person sex
(integer) 0
```

当字段不存在时赋值：HSETNX 命令与 HSET 命令类型，区别在于字段存在不做操作。

```
127.0.0.1:6379> HGET person name
"rcx1"
127.0.0.1:6379> HSETNX person name rcx2
(integer) 0
127.0.0.1:6379> HGET person name
"rcx1"
127.0.0.1:6379> HSETNX person sex man
(integer) 1
127.0.0.1:6379> HGET person sex
"man"
```

增加数字：HINCRBY 命令可以增加指定的整数。散列表没 HINCR 命令。

```
127.0.0.1:6379> HGET person age
"18"
127.0.0.1:6379> HINCRBY person age 1
(integer) 19
```

删除字段：HDEL 可以删除一个或者多个字段，返回值是被删除字段的个数。

```
127.0.0.1:6379> HDEL person sex
(integer) 1
127.0.0.1:6379> HDEL person sex
(integer) 0
127.0.0.1:6379> HGET person sex
(nil)
```

获取散列表的字段名和字段值：

```
127.0.0.1:6379> HKEYS person
1) "name"
2) "age"
127.0.0.1:6379> HVALS person
1) "rcx1"
2) "19"
```

获取字段数量：返回有多少个字段

```
127.0.0.1:6379> HLEN person
(integer) 2
```

【参考资料】

1. [Redis入门指南](http://book.douban.com/subject/24522045/)

---EOF---
