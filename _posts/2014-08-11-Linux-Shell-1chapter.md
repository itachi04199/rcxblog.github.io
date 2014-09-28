---
layout: post
title: linux shell 小试牛刀
categories: linux
tags: linux shell
---

#### 第一章 小试牛刀

##### 简介

- shell脚本通常是以#!开始的
- 有两种运行脚本的方式。一种是将脚本作为sh的命令行参数，另外一种是将脚本作为具有执行权限的可执行文件。

```
$ sh script.sh	#假设脚本位于当前目录 第一种方式。
为了让shell独立运行，需要可执行权限。
$ chmod a+x script.sh 修改完之后可以使用如下方式运行
./script.sh
shell程序读取首行，查看shebang行是否为#!/bin/bash，系统会识别/bin/bash，并且在内部以如下命令执行：
$ /bin/bash script.sh
```

##### 终端打印

- echo用于终端打印的基本命令。如：echo "hello"、echo  hello、echo 'hello'。但是这3种方式都有需要注意的地方：

1. echo不带引号的情况无法在所要显示的文本中使用，因为在bash shell当中被用作命令定界符。
2. echo hello;hello为例，echo hello被视为一个命令，第2个hello被视为另外一个命令。
3. 使用单引号echo的时候，Bash不会对单引号中的变量（$var）求值，只是原样显示。

- printf也可以用于终端打印，与C语言的printf函数一样。printf不会自动添换行。
 
