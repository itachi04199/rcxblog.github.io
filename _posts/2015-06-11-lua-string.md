---
layout: post
title: lua字符串学习
categories: lua
tags: lua, string
---

### 基础字符串函数

函数|含义
:--|:--
string.lower(s)|字符串转换成小写
string.upper(s)|字符串转换成大写
string.len(s)|字符串长度
string.rep(s,2)|将字符串 s 重复 2 次
string.sub(s, j, j)|截取字符串

### 模式匹配

#### string.find 函数

string.find 函数用于在给定的目标字符串当中搜索一个模式。例如：

```lua
s = "hello world"
i, j = string.find(s, "hello")
print(i, j) ---> 1  5
```

find 函数会返回匹配到的起始所以和结尾索引。如果没匹配到任何就返回 nil。

#### string.match 函数

string.match 返回的是目标字符串中与模式相匹配的那部分子串。

```lua
print(string.match("hello world", "hello")) ---> hello
```

#### string.gsub 函数

string.gsub 函数有三个参数，目标字符串、模式、和替换字符串。它基本的用法是将目标字符串中所有出现模式的地方替换为替换字符串。

```lua
s = string.gsub("lua is good", "good", "nice")
print(s) ---> lua is nice
```

另外还有第四个参数，可以限制替换的次数。

#### string.gmatch 函数

string.gmatch 函数会返回一个函数。通过这个函数可以遍历到一个字符串当中所有出现指定模式的地方。

```lua
words = {}

for w in string.gmatch(s, "%a+") do
	words[#words + 1] = w
end
```

### 模式

具体看下表：

字符|含义
:--|:--
.|所有字符
%a|字母
%c|控制字符
%d|数字
%l|小写字母
%p|标点符号
%s|空白字符
%u|大写字母
%w|字母和数字
%x|十六进制数字
%z|内部表示为0的字符

这些分类的大写形式表示他们的补集，例如，"%A"表示所有非字母字符。

在模式当中有如下元字符，需要进行转义。

```
( ) . % + - * ? [ ] ^ $
```

Lua 提供的修饰符：

字符|含义
:--|:--
+|重复一次或多次
*|重复零次或多次（尽可能多匹配）
-|重复零次或多次（尽可能少匹配)
?|出现零次或一次

### 捕获

捕获功能可以根据一个模式从目标字符串中抽取匹配该模式的内容，在指定捕获时，应该将模式中需要捕获的部分写到一对圆括号当中。

```lua
pair = "name=rcx"
key, value = string.match(pair, "(%a+)%s*=%s*(%a+)")
print(key, value) ---> name rcx
```

还可以对模式使用捕获。有一个例子，如果我们想再一个字符串当中找到由单引号或者双引号括起来的字符串。那么我们可以使用如下的模式：

```lua
"["'](.-)["']"
```

但是当字符串当中是 "it's all right" 这样的字符串就会出现问题。当然我们可以使用捕获第一个引号，然后用它来指定第二个引号：

```lua
s = [[then he said: "it's all right"]]
q, v = string.match(s, "([\"'])(.-)%1")
print(q)
print(v)
```

对于捕获到的值，还可以使用 gsub 函数的字符串替换。和模式一样，用于替换的字符串中可以包含 %d 这样的项。当进行替换的时候，这些项就对应于捕获到的内容。%0 表示整个匹配，并且替换字符串中的 % 必须进行转义到 %% 。

```lua
print(string.gsub("hello", "%a", "%0-%0"))
=> h-he-el-ll-lo-o	5
```

string.gsub 会返回两个值，第二个返回值表示被替换的次数

### 替换

string.gsub 函数的第三个参数不仅是一个字符串，也可以是一个函数或者 table。当传入的是函数的时候，string.gsub 会再每次找到匹配时候调用该函数，传入函数的值就是捕获到的内容，返回值作为替换的字符串。当传入的参数是 table 的时候，string.gsub 会每次捕获到内容作为 key 在 table 当中进行查找，并且将 value 作为要替换的字符串，如果 table 当中找不到值则不进行替换。

【参考资料】

1. [Lua程序设计](http://book.douban.com/subject/3076942/)

---EOF---
