---
layout: post
title: redis集合和有序集合类型
categories: redis
tags: redis
---

#### 集合类型

集合中的每一个元素都不同，并且没有顺序。一个集合类型键可以存储最多 2 的 32 次幂 - 1 个字符串。集合类型内部是使用散列表实现的。

增加/删除元素：

```
SADD key member [member ...]
SREM key member [member ...]
```

```
redis 127.0.0.1:6379> SADD letter a
(integer) 1
redis 127.0.0.1:6379> SREM letter a
(integer) 1
```

获取集合中所有元素：

```
redis 127.0.0.1:6379> SMEMBERS letter
1) "a"
2) "v"
```

判断元素是否在集合当中：在集合当中返回 1，不在返回 0

```
redis 127.0.0.1:6379> SISMEMBER letter a
(integer) 1
redis 127.0.0.1:6379> SISMEMBER letter b
(integer) 0
```

集合间运算：

```
SDIFF key [key ...]
SINTER key [key ...]
SUNION key [key ...]
```

SDIFF 命令原来对多个集合执行差集运算。A - B，代表所有属于 A 且不属于 B 的元素构成的集合。

```
redis 127.0.0.1:6379> SMEMBERS letter
1) "a"
2) "v"
redis 127.0.0.1:6379> SMEMBERS sett
1) "a"
2) "b"
redis 127.0.0.1:6379> SDIFF letter sett
1) "v"
redis 127.0.0.1:6379> SDIFF sett letter
1) "b"
```

如果是 SDIFF setA setB setC 计算顺序是先计算 setA - setB，在计算结果与 setC 的差集。

SINTER 命令原来计算多个集合交集。A 与 B 的交集是所有属于 A 并且属于 B 的元素组成的集合。

```
redis 127.0.0.1:6379> SMEMBERS letter
1) "a"
2) "v"
redis 127.0.0.1:6379> SMEMBERS sett
1) "a"
2) "b"
redis 127.0.0.1:6379> SINTER letter sett
1) "a"
```

SUNION 命令来执行集合的并集。集合 A 与集合 B 的并集是属于 A 或属于 B 的元素构成的集合。

```
redis 127.0.0.1:6379> SMEMBERS letter
1) "a"
2) "v"
redis 127.0.0.1:6379> SMEMBERS sett
1) "a"
2) "b"
redis 127.0.0.1:6379> SUNION letter sett
1) "a"
2) "b"
3) "v"
```

获得集合元素个数：

```
redis 127.0.0.1:6379> SCARD sett
(integer) 3
```

进行集合运算并且将结果存储：将运算的结果存储到 okey 当中

```
SDIFFSTORE okey key [key...]
SINTERSTORE okey key [key...]
SUNIONSTORE okey key [key...]
```

随机获得集合中元素：

```
redis 127.0.0.1:6379> SRANDMEMBER sett
"f,d"
redis 127.0.0.1:6379> SRANDMEMBER sett 2
1) "f,d"
2) "a"
```

可以传递 count 参数来随机获取多个元素：

- 当 count 是正数，会随机 count 个元素，如果大于集合个数，则返回全部元素
- 当 count 是负数，会获得 |count| 个元素

从集合中弹出一个元素：会随机从集合当中弹出一个元素

```
SPOP key
```

#### 有序集合

有序集合和列表有些类似：

- 都是有序的
- 都可以获取某一范围的元素

区别是：

- 列表是链表实现的，获取两端的数据极快，适合“新鲜事”或“日志”这样的应用
- 有序集合使用散列表实现，随机读取很快
- 列表不能简单的调整某个元素的位置，但是有序集合可以
- 有序集合更耗费内存

增加元素：分数可以是浮点数

```
redis 127.0.0.1:6379> ZADD tt 98 rcx
(integer) 1
redis 127.0.0.1:6379> ZADD tt 78 wx
(integer) 1
redis 127.0.0.1:6379> ZADD tt 85 rh
(integer) 1
```

获取元素的分数：

```
redis 127.0.0.1:6379> ZSCORE tt rcx
"98"
```

获得排名在某个范围的元素列表：

```
redis 127.0.0.1:6379> ZRANGE tt 0 2
1) "wx"
2) "rh"
3) "rcx"
```

ZRANGE 与 LRANGE 命令相似，索引都是从 0 开始，负数代表从后向前查找。

如果 ZRANGE 命令尾部加上 WITHSCORES 参数会返回元素的分数

```
redis 127.0.0.1:6379> ZRANGE tt 0 2 WITHSCORES
1) "wx"
2) "78"
3) "rh"
4) "85"
5) "rcx"
6) "98"
```

ZREVRANGE 与 ZRANGE 的区别是按照分数从大到小排列。

获得指定分数范围的元素：

```
ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT offset count]
```

```
redis 127.0.0.1:6379> ZRANGEBYSCORE tt 80 100
1) "rh"
2) "rcx"
```

如果不希望分数范围包含端点，可以在分数前加上 '(' 。

```
redis 127.0.0.1:6379> ZRANGEBYSCORE tt 80 (98
1) "rh"
```

WITHSCORES 的用法与 ZRANGE 相同，LIMIT offset count 是在获得的元素列表基础上向后偏移 offset 个元素，并且只获取 前 count 个元素。

增加某个元素的分数：

```
redis 127.0.0.1:6379> ZINCRBY tt 4 rh
"89"
```

获取集合中元素的个数：

```
redis 127.0.0.1:6379> ZCARD tt
(integer) 3
```

获取指定分数范围内的元素个数：

```
redis 127.0.0.1:6379> ZCOUNT tt 80 100
(integer) 2
```

删除元素：

```
redis 127.0.0.1:6379> ZREM tt rcx
(integer) 1
```

按照排名范围删除元素：

```
redis 127.0.0.1:6379> ZRANGE tt 0 -1
1) "wx"
2) "rcx"
3) "rcx1"
4) "rcx4"
5) "rh"
redis 127.0.0.1:6379> ZREMRANGEBYRANK tt 0 2
(integer) 3
redis 127.0.0.1:6379> ZRANGE tt 0 -1
1) "rcx4"
2) "rh"
```

按照分数范围删除元素：

```
redis 127.0.0.1:6379> ZRANGE tt 0 -1 WITHSCORES
 1) "rcx4"
 2) "83"
 3) "rh"
 4) "89"
 5) "rh1"
 6) "90"
 7) "rh2"
 8) "91"
 9) "rh3"
10) "93"
redis 127.0.0.1:6379> ZREMRANGEBYSCORE tt 90 93
(integer) 3
redis 127.0.0.1:6379> ZRANGE tt 0 -1 WITHSCORES
1) "rcx4"
2) "83"
3) "rh"
4) "89"
```

获得元素的排名：ZREVRANK 是按照分数从大到小顺序排名

```
redis 127.0.0.1:6379> ZRANK tt rh
(integer) 1
redis 127.0.0.1:6379> ZRANK tt rcx4
(integer) 0
```

#### 计算有序集合的交集

```
ZINTERSTORE dest numkeys key [key ...] [WEIGHTS weight [weight ...]] [AGGREGATE SUM|MIN|MAX]
```

ZINTERSTORE 命令计算多个有序集合的交集结果存储在 dest 当中，返回 dest 元素的个数。

dest 元素的分数是由 AGGREGATE 参数决定的。

当 AGGREGATE 是 SUM 时候，也就是默认值：

```
redis 127.0.0.1:6379> ZADD t1 1 a
(integer) 1
redis 127.0.0.1:6379> ZADD t1 2 b
(integer) 1
redis 127.0.0.1:6379> ZADD t2 10 a
(integer) 1
redis 127.0.0.1:6379> ZADD t2 11 b
(integer) 1
redis 127.0.0.1:6379> ZINTERSTORE t3 2 t1 t2
(integer) 2
redis 127.0.0.1:6379> ZRANGE t3 0 -1 WITHSCORES
1) "a"
2) "11"
3) "b"
4) "13"
```

当 AGGREGATE 是 MIN 时候，dest 元素的分数是计算集合中该元素分数的最小值。

当 AGGREGATE 是 MAX 时候，dest 元素的分数是计算集合中该元素分数的最大值。

【参考资料】

1. [Redis入门指南](http://book.douban.com/subject/24522045/)

---EOF---

