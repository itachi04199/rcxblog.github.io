---
layout: post
title: mysql 索引学习 上
categories: mysql
tags: mysql
---

### 索引类型

#### B-TREE 索引

如下例子：

```sql
CREATE TABLE user (
	last_name varchar(50) not null,
    first_name varchar(50) not null,
    dob date not null,
    gender enum('m', 'f') not null,
    key(last_name, first_name, dob)
);
```

上面是创建了一个三个字段的联合索引，这个索引对如下类型的查询有效：

- 全值匹配：即三个字段条件都用到
- 匹配最左前缀：即 last_name = 'rcx'
- 匹配列前缀：last_name like 'A%'
- 匹配范围值：last_name between 'Allen' and 'Barry'
- 精确匹配一列范围匹配另外一列：last_name = 'rcx' and first_name like 'L%'

B-TREE 索引有如下的一些限制：

- 如果不使用索引最左列查找，无法使用索引。
- 不能跳过索引中的列，如果指定 last_name = 'rcx' and dob = '2015-04-01'，则只能使用索引的第一列。
- 如果查询中某个列有范围查询，那么其右边所有列都无法使用索引优化查询。例如： last_name = 'rcx' and first_name like 'J%' and dob = '2015-01-04'，这个查询只可以用到索引的前两列。

#### 哈希索引

只有 Memory 引擎显示支持哈希索引。

### 索引的优点

在 B-TREE 索引中相关的列值也会存储在一起，所以某些查询只使用索引就能够完成全部查询。

1. 索引减少了服务器需要扫描的数据量。
2. 索引可以帮助服务器避免排序和临时表。
3. 索引可以将随机 I/O 变为顺序 I/O。

### 索引策略

- 查询的列不是独立的，则 mysql 无法使用索引。

```sql
select id from user where id + 2 = 5;//无法使用索引
```

- 多列索引

创建的错误就是为每一列创建索引。

例如，user 表当中在 name 和 age 字段上都单独有一个索引，那么在如下查询的时候会用不到索引(在 mysql 5.0 以前)：

```sql
select * from user where name='rcx' or age = 13;
```

这个时候 mysql 会优化查询，查询能同时进行单列索引扫描，将结果合并。

当出现索引合并的时候说明索引创建有些问题：

- 当查询出现多个 and 条件的时候，通常意味着需要一个包含所有相关列的索引，而不是多个独立的单列索引。
- 当条件出现多个 or 的时候，通常会耗费大量 cpu 和内存去进行合并操作。

通常这样的情况还不如使用如下 sql ：

```sql
select * from user where name='rcx' union all
select * from user where age = 13;
```

- 索引顺序

在一个多列 B-TREE 索引当中，索引列的顺序意味着索引首先按照最左列进行排序，其次是第二列等。

当不考虑排序和分组的时候，将选择性最高的列放在前面通常的很好的。当然，性能不只是依赖所有索引列的选择性，也和查询条件的具体值有关，也就是和值的分布有关。

看下面的例子：

```sql
select * from user where age = 15 and sex = 'man';
```

上面创建索引怎么创建？是(age,sex)还是(sex,age)？这个时候我们可以查询下数据的分布：

```sql
select sum(age=15),sum(sex='man') from user;
7222, 40
```

这个时候我们创建索引的时候就应该(sex,age)这样，因为 sex='man' 条件的数量更小。

当然这样选择索引顺序是依赖特定的数据值的，我们还是需要考虑全局基数和选择性。

### 覆盖索引

一般情况我们会根据查询的 where 条件来创建合适的索引，不过只是优化一个地方，如果通过索引来获取需要查询的数据，这样就不需要再读取数据行了。

在 mysql 当中只能使用 B-TREE 索引做覆盖索引。

当使用 explain 一个 sql 的时候，当 extra 列的值是 using index 的时候这个就是覆盖索引了。

一般索引无法覆盖有两个原因：

- 没有索引列覆盖到 select 查询的列当中
- mysql 不能在索引中执行 like 操作，但是 mysql 能在索引中做最左前缀匹配的 like 查询。

【参考资料】

1. [高性能 mysql] (http://book.douban.com/subject/4241826/)


