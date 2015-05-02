---
layout: post
title: mysql 一些知识点补漏
categories: mysql
tags: mysql
---

mysql 当中的 SHOW 命令：

```bash
SHOW DATABASES;// 返回可用数据库的列表
SHOW TABLES;// 返回数据库内的表的列表
SHOW COLUMNS FROM users;// 查看 users 表当中的字段
```

现在查询的结果，可以使用 limit：

```bash
select name from user limit 5;
select name from user limit 5, 5;
```

limit 5，代表返回的行不多于 5 行。
limit 5, 5，代表返回从第 5 行开始的 5 行。

在 mysql 当中可以使用 concat() 拼接字符串。

```bash
select concat(name, age, sex) from user;
```

在 mysql 当中有很多处理文本的函数：

函数|说明
:--|:--
Left()|返回字符串左边的字符
Length()|返回字符串的长度
Locate()|找出字符串的一个子串
Lower()|转换成小写
LTrim()|去掉字符串左边的空格
SubString()|返回字串的字符
Upper()|转换成大写

还有许多其他的内置函数。

---EOF---

