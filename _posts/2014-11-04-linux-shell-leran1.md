---
layout: post
title: 构建基本shell脚本
categories: linux 
tags: linux shell
---

#### 环境变量

shell维护着一组环境变量，用来记录系统特定的信息。如，系统名称，用户UID等。使用set命令查看完整的环境变量。

```bash
$ set
BASH=/bin/bash
LINES=39
TMOUT=1800
UID=40012
USER=chunxiao.ren
HOME=/home/chunxiao.ren
...
```

可以在shell脚本中使用系统的环境变量。使用方式如下：

```bash
#! /bin/bash
echo $PATH
echo $HOME
echo "this $PATH is a $15"
```

看上面最后一行，实际的想法是想输入$15这个字符，但是脚本会尝试显示变量$1(但是为定义)。想要显示美元符，需要在它前面放置一个反斜线。

```bash
echo "this $PATH is a \$15"
```

#### 用户变量

shell脚本中可以使用自定义变量。用户变量可以是任何不超过20个字母、数字或者下划线的字符串。用户变量区分大小写。

如下例子：

```bash
a=10
str="this is a string"
```

shell脚本自动决定变量的数据类型。在脚本的整个生命周期里，shell脚本中定义的变量会一直保存他们的值，但是在shell脚本执行完成时候删除掉。

用户变量也可以通过美元符引用：

```bash
#! /bin/bash
a=1
echo "this a is $a"
```

变量每次被引用的时候都会输出当前赋给它的值。引用变量值时需要使用美元符，对变量进行赋值时则不要使用美元符。

```bash
#! /bin/bash
value=10
value1=$value
```

如果对value1的赋值不加美元符则，value1的值就是value字符串了。

#### 反引号

反引号\`,允许将shell命令的输出赋值给变量。

```bash
#! /bin/bash
test=`date`
echo $test
```

#### 输出重定向

bash shell采用大于号(>)来完成这个功能：

```bash
date > a
```

重定向操作符创建了一个文件a，并且将date命令输出重定向到a文件中。如果文件存在了，则会覆盖这个文件的数据。

可以使用(>>)来在文件中追加数据。

#### 输入重定向

输入重定向跟输出重定向正好相反。输入重定向是将文件的内容重定向到命令。

输入重定向使用(<)。

```bash
grep hello < a
```

另外还有内联输入重定向，用(<<)来表示。

使用内联输入重定向的时候必须指定一个文本标记来划分要输入数据的开始和结尾。

```bash
grep hello <<EOF
heredoc> thisis
heredoc> hello
heredoc> EOF
```

#### 管道

有时候需要将某个命令的输入作为另外一个命令的输入。这个时候可以使用管道符（|）。

```bash
ps aux | grep java
```

#### expr命令

expr命令允许在命令行上处理数学表达式。

```bash
expr 1 + 5
expr 1 \* 5
```

运算符之间是带空格的，并且很麻烦，一般不使用。

#### 使用方括号

为了替代expr，更加简单的执行数学表达式。使用方式是$[operation]

```bash
a=$[1+2]
b=$[$a*2]
```

expr命令和方括号只支持整数运算。

#### 浮点运算

最常见的解决浮点运算的是bc。先输入bc命令，然后在提示符下输入浮点运算式。

#### 查看退出状态码

Linux提供了$?专属变量来保存上一个执行的命令的退出状态码。

```bash
$ date
Tue Nov  4 22:42:55 CST 2014
$ echo $?
0
```

按照惯例，一个成功结束的命令的退出状态是0.如果一个命令结束时有错误，退出状态中就会有一个正数值。

Linux退出状态码表如下：

状态码|描述
:--|:--
0| 命令成功结束
1| 通用未知错误
2| 误用shell命令
126| 命令不可执行
127| 没有找到命令
128| 无效退出参数
128+x| Linux信号x的严重错误
130| 命令通过Ctrl+C终止
255| 退出状态码越界

#### exit命令

默认情况，shell脚本会以脚本中最后一个命令的退出状态码退出。同时你也可以自定义退出返回状态码。

```bash
#! /bin/bash
var=1
exit 5
```

当执行完上面的脚本，并且查看$?的值的时候输出的是5。

---EOF---
