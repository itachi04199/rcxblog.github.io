---
layout: post
title: 代码中如何选取名字
categories: 代码优化
tags: 笔记 代码优化
---

我们在编写代码的时候有同一个主题思想，**代码应当易于理解**。

我们也可以说成**代码的写法应当使别人理解它所需的时间最小化**。

下面我们就来学习写代码表面层次上的改变。

#### 把信息装到名字里

- 选择专业的词
- 避免泛泛的名字
- 使用具体的名字代替抽象的名字
- 使用前缀或后缀给名字附带更多信息
- 决定名字的长度
- 利用名字的格式来表达含义

#### 选择专业的词

例如 'get' 这个词表达的信息就不是很多。不能明确的表示是从哪里获取。如下面的例子：

```java
void getPage(URL);
void fetchPage(URL);
void downLoadPage(URL);
```

上面的代码里面 fetchPage 和 downLoadPage 会比 getPage 表达的更加明确。

再看下面的例子：

```java
class BinaryTree {
	int size() {
    }
}
```

size 方法会返回什么呢，从名字看不出来，如果 size 换成 height 或者 numNodes 就很清晰的表达了从二叉树当中获取什么属性。

还有一个例子：

```java
class Thread {
	void stop() {
    }
}
```

stop 方法还可以，但是我们可以选择更加专业的名字，kill() 方法表达了重量级操作不可修复，也可以使用 pause() 方法或者 resume() 方法。

总体上可以说选择一个方法名称的时候，应该选择更加有表现力的词。

单词 | 更多选择
:-- | :--
send | deliver、dispath、announce、distribute
find | search、extract、locate、recover
start | launch、create、begin、open
make | create、set up、build、add、new

应该选用能**清晰**、**准确**表达方法意思的词汇。

#### 避免泛泛的名字

看下面的例子：

```java
var euclidean_norm = function (v) {
	var retval = 0.0
    for (var i=0; i < v.length; i++)
    	retval += v[i] * v[i];
    return Math.sqrt(retval);
}
```

上面这段代码描述的是累加 v 的平方。所以更加贴切的名字可以是 sum_squares。

当然有的时候也是需要泛泛的名字的。

下面举例泛泛名字的正确使用场景：

##### tmp

看下面经典代码：

```java
if (right < left) {
	tmp = right;
    right = left;
    left = tmp;
}
```

tmp 名字应用于短期存在且临时性的变量。

##### 循环迭代器

像 i,j,k 这样的变量在循环迭代当中我们会知道他是索引的意思，但是在复杂循环当中有的时候也会搞混。

如果把 i,j,k 换成 club_i,members_j,user_k 这样在循环的时候就不容易把这几个索引弄混乱。

#### 使用具体的名字代替抽象的名字

如果有个内部方法 serverCanStart()，它检测服务是否可以监听某个端口，那么这个名字不如 canListenOnPort() 更加的具体。

#### 为名字附带更多信息

如果有一个变量包含一个十六进制字符串：

```java
String id;// 例如： af84ec
//可以使用下面的名字
String hex_id;
```

如果参数返回是带单位的，要带上单位信息：

```java
var start = (new Date()).getTime();
//不如下面的好
var start_ms = (new Date()).getTime();
```

#### 名字的长度

在小作用域里可以使用短名字，因为作用域小就是在几行代码当中可见，所以变量的信息很容易看到。但是全局变量的名字就应该包含足够的信息。

为了缩减名字的长度不要乱用缩写。对程序员来说 eval 代替 evaluation，用 doc 代替 document，用 str 代替 string 是比较普遍的。

丢掉没用的词也可以减少名字的长度，convertToString() 就没有 toString() 简短，同样也可以使用 serveLoop() 来代替 doServeLoop()。

#### 避免有二义性的词

假设写一段操作数据库结果的代码：

```java
results = Database.all_objects.filter("year < 2011");
```

什么的代码有两种解释。

- 只包含年份小于2011年的结果
- 过滤掉年份小于2011年的结果

这里面的问题就是 filter 是有二义性的。我们看不清楚到底是挑出还是过滤。

#### 用 max 和 min 表示（包含）极限

```
int MAX_SIZE = 10;
if shopping.num_items() <= MAX_SIZE
```

#### 用 first 和 last 表示包含的范围

#### 用 begin 和 end 表示包含/排除范围

#### 给布尔值命名

通常来讲给布尔值变量名加上，is、has、can，这样的可以把布尔值变得更加明确。

#### 与使用者期望相匹配

在 java 代码中 get* 方法一般都是对 bean 的属性的获取方法，所以一些需要计算的方法要避免使用 get 开头。

---EOF---

