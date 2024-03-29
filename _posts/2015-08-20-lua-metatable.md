---
layout: post
title: lua 元表学习
categories: lua
tags: lua,
---

Lua 中的每个值都有一个元表。table 和 userdata 可以有各自独立的原表，其他类型的值则共享其类型所属的单一元素。Lua 在创建新的 table 的时候不会创建元表。

```lua
t = {}
print(getmetatable(t)) --> nil
```

可以使用 setmetatable 来修改或设置 table 的元表：

```lua
t1 = {}
setmetatable(t,t1)
print(getmetatable(t)) --> table: 0x7fc5e9c2f670
```

## 例子

假设用 table 来表示集合，并且有一些函数来计算集合的交集和并集。

```lua
Set = {}

-- 根据参数列表的值创建新的集合
function Set.new(t)
	local set = {}
    for _,v in ipairs(t) do
    	set[v] = true
    end
    return set
end

-- 集合的并集
function Set.union(a, b)
	local res = Set.new{}
    for v in pairs(a) do
    	res[v] = true
    end
    for v in pairs(b) do
    	res[v] = true
    end
    return res
end

-- 集合的交集
function Set.intersection(a, b)
	local res = Set.new{}
    for k in pairs(a) do
    	res[k] = b[k] -- 在 b 集合当中不存在会返回 nil
    end
    return res
end

function Set.tostring(set)
	local t = {}
    for v in pairs(set) do
    	t[#t + 1] = v
    end
    return "{" .. table.concat(t, ",") .. "}"
end

function Set.print(s)
	print(Set.tostring(s))
end
```

如果我们让 Set 可以使用 + 来进行集合的并集操作，那么我们需要把所有用于表示集合的 table 共享一个元表。代码修改如下：

```lua
local mt = {}
function Set.new(t)
	local set = {}
    setmetatable(set, mt)
    for _,v in ipairs(t) do
        set[v] = true
    end
    return set
end
```

然后给元表添加方法，元方法 __add 是用于描述如何完成加法的。

```lua
mt.__add = Set.union
```

然后测试下：

```lua
local s1 = Set.new{10,20,30}
local s2 = Set.new{40,20,30}
local s3 = s1 + s2
Set.print(s3) --> {30,10,20,40}
```

还可以把 * 设置成交集：

```lua
mt.__mul = Set.intersection
```

在元表当中还有其他的方法：

__sub：减法

__div：除法

__unm：相反数

__mod：取模

__pow：乘幂

当两个集合相加，可以使用任意一个集合的元表，然而，当一个表达式中混合了具有不同元表的表达式，例如：

```lua
s = Set.new{1, 2, 3}
s = s + 8
```

Lua 会按照如下步骤查找元表，如果第一个值有元表，并且元表中有 __add 字段，那么 Lua 就以这个字段为元方法，而与第二个值无关。如果第一个没有，而第二个有就以第二个为准。如果两个都没有元方法，Lua 就会报错。

## 关系类的元方法

元方法还有 __eq（等于）、__lt（小于）、__le（小于等于）。

与算术元方法不一样的是，关系类的元方法不能用在混合的类型当中。如果将一个字符串与一个数字作顺序比较，Lua 会报错。同样，如果试图比较两个具有不相同元方法的对象，那么 Lua 也报错。

等于比较不会引发错误。但是如果两个对象有不同的元方法，那么等于操作不会调用任何一个元方法，而是直接返回 false。

## 库定义元方法

在程序库当中会定义自己的元表字段。函数 tostring 是一个例子，在 print 的时候，会默认调用 tostring 类格式化其输出。

所以上面的例子也可以修改：

```lua
mt.__tostring = Set.tostring
local s1 = Set.new{1,4, 2}
print(s1) --> {1,4,2}
```

函数 setmetatable 和 getmetatable 也会用到元表当中的一个字段，用于保护元表。

```lua
mt.__metatable = 'not your business'
```

如上设置后就不可以 setmetatable，并且 getmetatable 会返回设置的字符串。

## table 访问的元方法

### __index 元方法

当访问一个 table 当中不存在的字段时候，会返回 nil。如果我们设置了 table 的元表的 __index 方法，那么就会由这个方法提供最终的结果。

例子：

```lua
Window = {}
Window.prototype = {x=0,y=0,width=100,height=100}
Window.mt = {}

function Window.new(o)
	setmetatable(o, Window.mt)
    return o
end

Window.mt.__index = function(table, key)
	return Window.prototype[key]
end

w = Window.new{x=10,y =20}
print(w.width) --> 100
```

当 Lua 检测到 w 中没有某个字段，并且在元表中有一个 __index 字段，那么 Lua 就会以 w(table) 和 width(不存在的 key) 来调用这个 __index 方法。

__index 元方法不一定是一个函数，也可以是一个 table，所以前面的例子可以修改成如下：

```lua
Window.mt.__index = Window.prototype
```

现在就是如果 Lua 查找到元表的 __index 是一个 table，那么 Lua 就会在这个 table 中继续查找。

### __newindex 元方法

__newindex 元方法是用在赋值的，当对一个 table 当中不存在的索引赋值时，解释器会找到 __newindex 方法，如果有就执行，而不执行赋值操作。如果这个元方法是个 table，那么解释器就在此 table 上赋值，而不是原来的 table。

### 具有默认值的 table

普通的 table 任何字段的默认值都是 nil。也可以通过元表来修改这个默认值：

```lua
function setDefault(table, value)
	local mt = {__index = function() retuen value end}
    setmetatable(table, mt)
end

tab = {x=10, y=20}
print(tab.x, tab.z) --> 10   nil
setDefault(tab, 0)
print(tab.x, tab.z) --> 10   0
```

### 只读 table 的实现

```lua
function readOnly(t)
	local proxy = {}
    local mt = {
    	__index = t,
        __newindex = function (t, k, v)
        	error("this is a readonly table")
        end
    }
    setmetatable(proxy, mt)
   	return proxy
end
```
【参考资料】

1. [Lua程序设计](http://book.douban.com/subject/3076942/)

---EOF---
