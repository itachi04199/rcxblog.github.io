---
layout: post
title: 把控制流变得易读
categories: 代码优化
tags: 笔记 代码优化
---

## 第七章 把控制流变得易读

### 1.条件语句中的参数的顺序
比较的左侧放置倾向于变化的，比较的右侧是放置不变化的

```
 if(age > 18) 比 if(18 < age) 更加清晰
```

### 2.if/else语句块的顺序

虽然if/else语句的顺序可以随意调换，但是遵循下面的规则会使代码更加清晰：

- 首先处理正逻辑，不先处理负逻辑。如if(debug) 好于 if(!debug)。
- 先处理简单的逻辑。
- 先处理有趣或者可疑的情况。

### 3.避免使用三目运算符
所有的三目运算符形式都可以写成if/else形式。

**关键思想：只有当最简单的情况可以使用三目运算符。**
### 4.避免使用do、while循环

### 5.从函数中提前返回
编写如下代码：

```
 public boolean constains(String string,String subStr) {
     if(string == null || subStr == null ){
         return false;
        }
     if(string.equals(subStr)) {
         return true;
        }
    }
```

### 6.最小化嵌套
嵌套很深的代码很难理解，如下代码：

```
 if(userResult == SUCCESS) {
        if(permissonResult != SUCCESS) {
         reply.write("permissonResult error");
         reply.done();
         return;
        }
     reply.write("");
    } else {
     reply.write(userResult);
    }
 reply.done();
```

最初代码是简单的：

```
 if(userResult == SUCCESS) {
     reply.write("");
    } else {
     reply.write(userResult);
    }
 reply.done();
```

但是后来又添加个一个新的逻辑，代码变成了下面这样：

```
 if(userResult == SUCCESS) {
        if(permissonResult != SUCCESS) {
         reply.write("permissonResult error");
         reply.done();
         return;
        }
     reply.write("");
    } else {
     reply.write(userResult);
    }
 reply.done();
```

**关键思想：当你对代码修改时，从全新的角度来审视，从整体上来看修改后的影响。**

可以通过提早返回来减少嵌套：

```
 if(userResult != SUCCESS) {
     reply.write(userResult);
        reply.done();
        return;
    }
    if(permissonResult != SUCCESS) {
        reply.write("permissonResult error");
        reply.done();
        return;
    }
    reply.write("");
 reply.done();
```

如何减少循环内的嵌套，在循环中可以使用continue来进实现类似提前返回的效果

```
 for(int i = 0; i < results.size(); i ++){
     if(results[i] == null) continue;
     count++;
        if(results[i].getName().equals("")) continue;
        print(results[i].getName());
    }
```

### 总结
- 写一个比较的时候，把改变的值写在左边并且把更稳定的值写在右侧。
- 重新排列if/else语句。通常来讲，先处理正确的、简单的、有趣的情况。
- 避免使用三目运算符、do/while循环。
- 减少代码的嵌套。尽量写的更加线性。

