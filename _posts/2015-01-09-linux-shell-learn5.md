---
layout: post
title: shell 处理文件描述符
categories: linux
tags: linux shell
---

#### 标准文件描述符

Linux 用文件描述符来标识每一个文件对象。文件描述符是一个非负整数，可以唯一地标识回话中打开的文件。bash shell 保留了 3 个文件描述符。

文件描述符|缩写|描述
:--|:--|:--
0|STDIN|标准输入
1|STDOUT|标准输出
2|STDERR|标准错误

STDIN 文件描述符代表 shell 的标准输入，对于终端来说，标准输入就是键盘。

在使用输入重定向符号 (<) 时，Linux 会用重定向指定的文件来替换标准输入文件描述符。

STDOUT 文件描述符代表标准的 shell 输出。在终端上，标准输出就是终端显示器。

使用输出重定向符号 (>、>>)，可以将要输出到显示上的内容重定向到指定的文件中。

STDERR 文件描述符用来处理错误消息。它代表 shell 的标准错误输出。

默认情况下 STDOUT 和 STDERR 指向同样的地方，默认情况下，错误消息也会输出到显示器输出中。

#### 重定向错误

只重定向错误，如下：在上面的表中看到 STDERR 文件描述符被设置成 2。

```
$ ls t 2> error
$ cat error
ls: cannot access t: No such file or directory
```

重定向错误和数据，如下：

```
$ ls task t 2> error 1> list
$ cat list
task
$ cat error
ls: cannot access t: No such file or directory
```

也可以将 STDOUT 和 STDERR 输出到同一个文件。

```
$ ls task t &> out
$ cat out
ls: cannot access t: No such file or directory
task
```

#### 在脚本中重定向输出

有两种方式在脚本中重定向输出：

- 临时重定向每行输出
- 永久重定向脚本中的所有命令

**临时重定向**

如果你要故意在脚本中生成错误消息，需要将单独的一行输出重定向到 STDERR。如下：

```
$ echo "error msg" >&2
```

在重定向到文件描述符时，你必须在文件描述符数字和输出重定向符号之间加上一个 `&`。

如果执行如下脚本：

```
$ cat test.sh
#! /bin/bash
echo "error msg" >&2
echo "normal msg"

$ ./test.sh
error msg
normal msg
```

默认情况下 Linux 会将 STDERR 定向到 STDOUT。但是，如果你在运行脚本时重定向了 STDERR，脚本中所有定向到 STDERR 的文本都会被重定向：

```
$ ./test.sh 2> test
normal msg
$ cat test
error msg
```

**永久重定向**

如果在脚本中有大量数据需要重定向，可以使用 exec 命令告诉 shell 在脚本执行期间重定向到某个特定文件描述符：

```
$ cat test.sh
#! /bin/bash
exec 1>testout

echo "error msg2"
echo "normal msg2"
echo "error msg1"
echo "normal msg1"
$ ./test.sh
$ cat testout
error msg2
normal msg2
error msg1
normal msg1
```

#### 在脚本中重定向输入

可以使用 exec 命令将 STDIN 重定向到 Linux 系统上的文件中：

```
exec 0< testfile
```

这个命令告诉 shell 从文件 testfile 中获得输入，而不是 STDIN。

---EOF---

