---
layout: post
title: shell当中的if-else语句
categories: linux
tags: linux shell
---

#### if-then语句

if-then语句的格式如下：

```bash
if command
then
	commands
if
```

bash shell的if语句会执行if行定义的那个命令，如果该命令正常退出(退出状态码是0)，则then部分的命令就会执行。

例如:

```bash
#! /bin/bash
if date
then
	echo "asd"
fi
```

这个例子，会先执行date命令，如果date命令执行成功，则会执行echo命令，所以运行结果如下：

```bash
Wed Nov  5 19:54:38 CST 2014
asd
```

if-then语句也有如下的形式：

```bash
if command; then
	commands
fi
```

if后面的命令加上一个分号，这样then可以与if在同一行。

#### if-then-else语句

if-then-else的语句结构如下：

```bash
if command
then
	commands
else
	commands
fi
或者
if command; then
	commands
else
	commands
fi
```

#### 嵌套if

嵌套if的语句结构如下：

```bash
if command1
then
	commands
elif command2
then
	commands
fi
或者
if command1; then
	commands
elif command2; then
	commands
fi
```

#### test命令

test命令提供了在if-then语句当中测试不同条件的途径。使用方式如下：

```bash
if test command
then
	commands
fi
```

bash shell 提供了另外一种if-then语句中声明test命令的方法：

```bash
if [ condition ]
then
	commands
fi
```

方括号当中是条件，并且条件和方括号之间有一个空格。

test命令可以判断3类条件：

- 数值比较
- 字符串比较
- 文本比较

##### 数值比较

数值比较功能表如下：

比较|描述
:--|:--
n1 -eq n2| n1 == n2
n1 -ge n2| n1 >= n2
n1 -gt n2| n1 > n2
n1 -le n2| n1 <= n2
n1 -lt n2| n1 < n2
n1 -ne n2| n1 != n2

数值条件测试可以用在数字和变量上。例如：

```bash
a=2
if [ $a -gt 5 ]
```

##### 字符串比较

test命令也可以比较字符串值。如下所示：

比较|描述
:--|:--
str1 = str2| 比较相等
str1 != str2| 比较不相等
str1 > str2| str1大于str2
str1 < str2| str1小于str2
-n str1| 检查str1的长度是否非0
-z str1| 检查str1的长度是否为0

在字符串比较大小的时候 `>` 和 `<` 必须经过转义。

```bash
str1=ad
str2=we
if [ $str1 \> $str2 ]
```

sort命令处理大写字幕的方法正好跟test命令的相反。

```bash
#! /bin/bash
s1=Test
s2=test
if [ $s1 \> $s2 ]; then
 echo "s1 > s2"
else
 echo "s1 < s2"
fi

> ./te.sh #执行脚本
s1 < s2
```

但是sort排序的时候test会出现在Test上面。

当空串和未初始化的变量在`if [ -n "$var" ]`的时候都不会执行它的then语句，但是会执行`if [ -z "$var" ]`的then语句。

##### 文件比较

test命令允许你测试Linux系统上文件和目录的状态。如下表：

比较|描述
:--|:--
-d file| 检查file是否存在并且是一个目录
-e file| 检查file是否存在
-f file| 检查file是否存在并且是一个文件
-r file| 检查file是否存在并且可读
-s file| 检查file是否存在并且非空
-w file| 检查file是否存在并且可写
-x file| 检查file是否存在并且可执行
-O file| 检查file是否存在并且所有者是当前用户
-G file| 检查file是否存在并且所属组与当前用户相同
file1 -nt file2| 检查file1是否比file2新
file1 -ot file2| 检查file1是否比file2旧

#### 复合条件测试

if-then语句当中允许使用布尔逻辑来组合测试。有两种情况：

- [ condition1 ] && [ condition2 ]
- [ condition1 ] || [ condition2 ]

```bash
if [ condition1 ] && [ condition2 ]
then
	commands
fi

if [ condition1 ] || [ condition2 ]
then
	commands
fi
```

#### if-then高级特性

- 用于数学表达式的双小括号
- 用于高级字符串处理功能的双方括号

使用双小括号格式如下：

```bash
(( expression ))
```

支持的运算如下表：

符号|描述
:--|:--
val++|后增
val--|后减
++val|前增
--val|前减
!|逻辑求反
~|位求反
**|幂运算
<<|左移位
>>|右移位
&|位运算与
一个竖线 |位运算或
&&|逻辑与
两个竖线|逻辑或

具体使用如下：

```bash
val=10

if (( $val ** 2 > 90 ))
then
	commands
fi
```

并且在双小括号里面的大于号不需要转义。

使用双方括号提供了针对字符串比较的高级特性，使用方式如下：

[[ expression ]]

它提供了一个模式匹配的功能，可以定义正则表达式来匹配字符串。

```bash
if [[ $USER == chun* ]]
then
	echo "hello $USER"
fi
```

#### case命令

case的命令格式如下:

```bash
case var in
pattern1 | pattern2) command1;;
pattern3) command2;;
*) defaultCommands;;
esac
```

下面是个简单的例子：

```bash
v=chunxiao

case $v in
  chun | asew)
        echo "a";;
  asdd2)
        echo "b";;
  *)
        echo "c";;
esac
```

---EOF---

