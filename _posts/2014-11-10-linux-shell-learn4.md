---
layout: post
title: shell当中处理用户输入
categories: linux
tags: linux shell
---

#### 命令行参数

向shell脚本传递参数的最基本方式就是使用命令行参数。

```baah
$ ./a.sh 10 30
```

可以使用如下方式来读取传入的参数。这些参数被成为位置参数。

$0是脚本名称，$1是第一个参数，$2是第二个参数，以此类推一直到$9。如果参数的个数大于9个，则需要${10}这样来获取传入的参数值。

当给$0变量的真实字符串是整个脚本的路径时，程序中就会使用整个路径，而不仅仅是程序名。

可以使用basename命令只会返回程序名而不包含路径。

```bash
name=`basename $0`
echo $name

$  >/home/chunxiao/shell/te.sh
te.sh
$  >./te.sh
te.sh
```

如果脚本当中使用了命令行参数，但是在执行脚本的时候没有进行传入参数，这个时候就会程序报错。所以在使用命令行参数的时候，事先判断下是很有必要的。

```bash
if [ -n $1 ]; then
	command
fi
```

#### 特殊参数变量

$#特殊变量代码命令行参数的个数。

如果要获取最后一个命令行参数的值可以使用${$#}，但是很遗憾这样是不正确的，因为在{}里面是不可以使用$符号，但是可以写成${!#}。

$*和$@代表了所有传入的命令行参数。

但是两者有点小小的区别。

```bash
for v in "$@"; do
  echo $v
done

for v in "$*"; do
  echo "*:$v"
done
#输出结果为
a
b
c
*:a b c
```

使用for循环遍历这两个特殊变量，可以看到他们处理方式的不同。$*变量将所有参数当成了一个参数，$@将会单独处理每个参数。

#### 移动变量

在bash shell当中可以使用shift命令来操作命令行参数。

在使用shift命令时，默认情况会将每个参数变量减一。所以$3的值会移动到$2，$2的值会移动到$1，原来的$1值会被删掉，$0的值是不会变动的。

#### 处理选项

grep命令会有-i，-v等选项。在脚本中也可以获取这些选项。

如果想在shell脚本中同时使用选项和参数。linux中处理这个问题的标准方式是用特殊字符来将二者分开，该字符会告诉脚本选线何时结束以及普通参数何时开始。对于linux来说，这个特殊字符是`--`。

##### 使用getopt命令：

getopt命令是一个在处理命令行选项和参数时非常方便的工具。能够识别命令行参数。格式如下：

```bash
getopt options optstring parameters
```

optstring定义了命令行有效的选项字母，还定义了那些选项字母需要参数。

首先，在optstring中列出你要在脚本中用到的每个命令行选项字母。然后，在每个需要参数值的选项字母后加一个冒号。

下面是个简单的例子：

```bash
$ getopt ab:cd -a -b test -c -d test1 test2
 -a -b test -c -d -- test1 test2
```

什么例子定义了4个有效选项字母，a、b、c和d。还定义了b需要一个参数值。

但是getopt命令不擅长处理带空格参数值。它会将空格当作参数分隔符，而不是根据双引号将二者做为一个参数。

##### getopts命令

getopts命令每次调用它时，它值处理一个命令行上检测到的参数。处理完所有参数后，会退出返回一个大于零的退出状态码。

格式如下:

```bash
getopts optstring variable
```

optstring和getopt命令中的那个类似。

创建shell脚本的时候，使用哪些字母选项完全取决于你自己，但是有写字母选项在Linux当中已经成为了标准。如果使用这些选项，脚本看起来会更加友好。

选项|描述
:--|:--
-a| 显示所有对象
-c| 生成一个技术
-d| 指定一个目录
-e| 扩展一个对象
-f| 指定读入数据的文件
-h| 显示命令的帮助信息
-i| 忽略文本大小写
-l| 产生输出的长格式版本
-n| 使用非交互模式
-o| 指定叫所有输出重定向到的输出文件
-q| 以安静模式运行
-r| 递归处理目录和文件
-s| 以安静模式运行
-v| 生成详细输出
-x| 排除某个对象
-y| 对所有问题回答yes

#### 获得用户输入

read命令接受从标准输入或另一个文件描述符的输入。如下示例：

```bash
echo "enter your name"
read name
echo "hello $name"
```

read命令包含的-p选项，允许直接在read命令行指定提示符。

```bash
read -p "enter your name" name
echo "hello $name"
```

你也可以在read命令行中不指定变量。这样read命令会把它接受到的任何数据都放进特殊环境变量REPLY中。

read命令可以指定等待的秒数，通过-t选项，当计时器过期后read命令会返回一个非零退出状态码。

```bash
if read -t 5 -p "your name is : " name
then
	echo "hello $name"
else
	echo "too slow"
if
```

也可以让read命令来对输入的字符个数进行限制，如果输入的字符达到预设的字符数时，它会自动退出。可以使用-n选项加上数字限制。

```bash
read -n1 -p "do you want to continue [Y/N]" answer
case $answer in
Y | y)
	echo "continue";;
N | n)
	echo "exit";;
*)
	echo "input error";;
esac
```

有时候需要读取用户的输入，但是不想输入出现在屏幕上。典型的例子就是输入密码，这个时候可以使用-s选项阻止传给read命令的数据显示在显示器上（实际上，数据会被显示，只是read命令会将文本颜色设置成跟背景色一样）。

```bash
read -s -p "enter your password" password
```

也可以通过read命令来读取Linux系统上文件里面的数据。每次调用read命令会从文件读取一行。当文件没内容的时候，read命令会返回非零退出状态码。

常见的方式是将文件cat之后传给read命令。如下示例：

```bash
cat a.txt | while read line
do
	echo $line
done
```

---EOF---

