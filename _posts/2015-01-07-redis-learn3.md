---
layout: post
title: redis列表类型
categories: redis
tags: redis
---

#### 列表类型

列表类型 (list) 可以存储一个有序的字符串列表，常用的操作是向列表两端添加元素，或者活的列表的某个片段。

列表内部是使用双向链表实现的，所以获取头部和尾部的记录是非常快的，向列表两端添加元素也非常快。不过通过索引访问元素比较慢。

向列表两端添加元素：命令返回值表示增加元素后列表的长度。

```
LPUSH key value [value ...]
RPUSH key value [value ...]
```

```
127.0.0.1:6379> LPUSH list a b c
(integer) 3
127.0.0.1:6379> RPUSH list e f
(integer) 5

//此时列表的位置如下
c b a e f
```

从列表两端弹出元素：

```
127.0.0.1:6379> LPOP list
"c"
127.0.0.1:6379> RPOP list
"f"
//此时列表的值如下
b a e
```

获取列表中元素的个数：当键不存在返回 0。

```
127.0.0.1:6379> LLEN list
(integer) 3
```

获得列表片段：列表的索引是从 0 开始，LRANGE 命令不会删除元素，LRANGE 命令返回的元素包括最右边的元素。

```
127.0.0.1:6379> LRANGE list 0 2
1) "b"
2) "a"
3) "e"

127.0.0.1:6379> LRANGE list -2 -1
1) "a"
2) "e"
```

LRANGE 命令也支持负索引，-1 表示右边第一个元素，-2 表示右边第二个元素。

显然 `LRANGE list 0 -1` 可以获取列表当中所有元素。

如果 start 的索引位置比 stop 的索引位置靠后，则会返回空列表。

如果 stop 大于实际的索引范围，则会返回到列表最右边的元素。

```
127.0.0.1:6379> LRANGE list 2 1
(empty list or set)
127.0.0.1:6379> LRANGE list 0 222
1) "b"
2) "a"
3) "e"
```

删除列表中指定的值的命令为：`LREM key count value`

LREM 命令会删除列表中前 count 个值为 value 的元素，返回值是实际删除的元素个数。根据 count 值的不同， LREM 命令的执行方式会略有差异：

- 当 count 〉0 ，LREM 命令会从列表左边开始删除前 count 个值为 value 的元素
- 当 count 〈 0 ，LREM 命令会从列表右边开始删除前 |count| 个值为 value 的元素
- 当 count = 0 ， LREM 命令会删除所有值为 value 的元素

```
127.0.0.1:6379> LRANGE list 0 -1
1) "b"
2) "e"
3) "e"
4) "a"
5) "e"
6) "f"
127.0.0.1:6379> LREM list 4 e
(integer) 3
127.0.0.1:6379> LRANGE list 0 -1
1) "b"
2) "a"
3) "f"
```

获得/设置指定索引是元素值：

```
127.0.0.1:6379> LRANGE list 0 -1
1) "b"
2) "a"
3) "f"
127.0.0.1:6379> LINDEX list 2
"f"
127.0.0.1:6379> LSET list 2 d
OK
127.0.0.1:6379> LINDEX list 2
"d"
```

只保留列表指定片段：LTRIM 命令删除指定索引之外的所有元素

```
127.0.0.1:6379> LRANGE list 0 -1
1) "b"
2) "a"
3) "d"
4) "v"
5) "m"
127.0.0.1:6379> LTRIM list 0 1
OK
127.0.0.1:6379> LRANGE list 0 -1
1) "b"
2) "a"
```

向列表中插入元素：`LINSERT key BEFORE|AFTER pivot value`，LINSERT 命令会首先在列表中从左到右查找值为 pivot 的元素，然后根据 BEFORE 或者 AFTER 决定在上面地方插入。

```
127.0.0.1:6379> LRANGE list 0 -1
1) "b"
2) "a"
127.0.0.1:6379> LINSERT list BEFORE a v
(integer) 3
127.0.0.1:6379> LRANGE list 0 -1
1) "b"
2) "v"
3) "a"
127.0.0.1:6379> LINSERT list AFTER a W
(integer) 4
127.0.0.1:6379> LRANGE list 0 -1
1) "b"
2) "v"
3) "a"
4) "W"
```

将元素从一个列表转移到另外一个列表：

```
127.0.0.1:6379> RPOPLPUSH list list1
"W"
127.0.0.1:6379> LRANGE list1 0 -1
1) "W"
127.0.0.1:6379> LRANGE list 0 -1
1) "b"
2) "v"
3) "a"
```

【参考资料】

1. [Redis入门指南](http://book.douban.com/subject/24522045/)

---EOF---
