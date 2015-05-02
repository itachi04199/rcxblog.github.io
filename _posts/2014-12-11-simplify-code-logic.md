---
layout: post
title: 简化代码逻辑
categories: 代码优化
tags: 笔记 代码优化
---

#### 拆分超长的表达式

如果代码块特别大的话会非常难以阅读，经常用的方法就是把超长表达式拆分成容易理解的小块。

看下面的例子：可以将表达式赋值给一个额外的变量。

```java
if (line.split(":")[0].trim().equals("root"))
 ...
```

可以修改成下面：

```java
String username = line.split(":")[0].trim();
if (username.equals("root"))
 ...
```

在看下面的例子：

```java
if (request.user.id == document.owner_id) {
 ...
}

if (request.user.id != document.owner_id) {
 ...
}
```

上面例子的 request.user.id == document.owner_id 表达式看起来不是很长，但是包含了5个变量，其实这段代码的意思就是判断该用户是否拥有此文档。所以可以这样修改：

```java
final boolean user_owns_document = (request.user.id == document.owner_id);

if (user_owns_document) {
 ...
}

if (!user_owns_document) {
 ...
}
```

#### 使用德摩根定理

对于布尔表达式有两种等价的写法：

- not (a or b or c) = (not a) and (not b) and (not c)
- not (a and b and c) = (not a) or (not b) or (not c)

所以有时候可以使用法则来改写布尔表达式，使得更加可读。

```java
if (!(file_exists && !is_protected)) {...}
//改成
if (!file_exists || is_protected)) {...}
```

看下面的例子：

```java
class Range {
 int begin;
    int end;
    boolean overLapsWith(Range other) {
     ...
    }
}

```

Range 类表示的是区间，是个左闭右开的区间。所以 overLapsWith 方法是检查是否自身范围的任意一个端点在 other 的范围之内。

```java
	boolean overLapsWith(Range other) {
     	return (begin >= other.begin && begin < other.end) ||
        			(end > other.begin && end <= other.end) ||
                    	(begin <= other.begin && end >= other.end);
    }
```

上面的实现方式看起来很复杂，我们可以使用更加优雅的方式来实现这个方法。我们可以判断这两个范围是否不重叠更简单，因为只有两种情况：

- 另一个范围在这个范围开始前结束
- 另一个范围在这个范围结束后开始

所以代码可以变成这样：

```java
	boolean overLapsWith(Range other) {
    	if ((begin >= other.end) || (end <= other.begin) ) {
        	return false;
        }
     	return true;
    }
```

#### 拆分巨大的语句

看下面代码:

```javascript
var update_highlight = function (message_num) {
	if($("#vote_value" + message_num).html() === "Up") {
    	$("#up" + message_num).addClass("highlighted");
        $("#down" + message_num).removeClass("highlighted");
    } else if ($("#vote_value" + message_num).html() === "Down") {
    	$("#up" + message_num).removeClass("highlighted");
        $("#down" + message_num).addClass("highlighted");
    } else {
    	$("#up" + message_num).removeClass("highlighted");
        $("#down" + message_num).removeClass("highlighted");
    }
}
```

看到上面的例子当中，很多表达式都是一样的，这意味着可以把它们提取出来作为函数开头的总结变量：

```javascript
var update_highlight = function (message_num) {
	var up = $("#up" + message_num);
    var down = $("#down" + message_num);
    var vote_value = $("#vote_value" + message_num);
	if(vote_value.html() === "Up") {
    	up.addClass("highlighted");
        down.removeClass("highlighted");
    } else if (vote_value.html() === "Down") {
    	up.removeClass("highlighted");
        down.addClass("highlighted");
    } else {
    	up.removeClass("highlighted");
        down.removeClass("highlighted");
    }
}
```

对于变量的草率运用会让程序更加难以理解。

- 变量越多，就越难全部跟踪它们的动向。
- 变量的作用域越大，就需要跟踪它的动向越久。
- 变量改变的越频繁，就越难以跟踪它的当前值。

#### 减少没价值的临时变量

看下面这段代码：

```python
now = datetime.datetime.now()
root_message.last_view_time = now
```

我们来看下 now 这个临时变量，它没有拆分任何复杂的表达式，也没有表达更多的含义，并且这个变量只用到了一次。所以代码修改成下面这样也很清晰：

```python
root_message.last_view_time = datetime.datetime.now()
```

那么看下面这个 js 函数，从数组当中删除一个值：

```javascript
var remove_one = function (array, value_to_remove) {
	var index_to_remove - null;
    for (var i = 0; i < array.length; i++) {
    	if (array[i] == value_to_remove) {
        	index_to_remove = i;
            break;
        }
    }

    if (index_to_remove != null) {
    	array.splice(index_to_remove, 1);
    }
};
```

变量 index_to_remove 只用来了保持临时结果。所以可以如下简化：

```javascript
var remove_one = function (array, value_to_remove) {
    for (var i = 0; i < array.length; i++) {
    	if (array[i] == value_to_remove) {
        	array.splice(i, 1);
            return;
        }
    }
};
```

#### 减少控制流变量

有些时候会看到下面这样的代码：

```java
boolean done = false;
while ( condition && !done) {
	...
    if (...) {
    	done = true;
        continue;
    }
}
```

像这样的代码我们应该尽量去避免而且可以运用结构化编程而消除。

```java
while ( condition ) {
    if (...) {
		break;
	}
}
```

#### 缩小变量的作用域

让你的变量对尽量少的代码行可见。

同时只写一次的变量更好，如 java 当中的 final 变量，这样的是不需要读者思考很多的。

---EOF---
