---
layout: post
title: shell当中的循环
categories: linux
tags: linux shell
---

#### for循环

for循环的基本格式如下：

```bash
for var in list
do
	commands
done
或者
for var in list; do
	commands
done
```

例如：

```bash
for var in a b c d; do
	echo this is $var
done
echo $var
```

输出完成后var会保存在后一次迭代的值。

如何读取列表中的复杂值。

```bash
for word in I don't know if this'll work
do
	echo "word:$word"
done
```

这个shell的输出会丢失掉单引号。

有两种解决办法：

- 使用转义字符将单引号转义
- 使用双引号来定义用到单引号的值

如下：

```bash
for word in I don\'t know if "this'll" work
do
	echo "word:$word"
done
```

for循环当中的list都用空格分割的，如果想在数据中包含空格就会出现问题。所以你需要将那些值用双引号引用起来，如下：

```bash
for word in "I want" "don't" "know if"
do
	echo "word:$word"
done
```

for循环也可以在变量中读取列表。

```bash
list="a b c d"
for word in $list
do
	echo "word:$word"
done
```

当然for循环也可以从命令读取值，如下：

```bash
for v in `echo a b c`
do
	echo $v
done
```

在for循环当中也可以更改字段分隔符。默认情况下，bash shell会将下列字符当作字段分隔符：

- 空格
- 制表符
- 换行符

如果想要改变分隔符，需要临时更改IFS环境变量的值。比如，如果想只是换行当作分隔符，需要如下操作：

```bash
IFS=$'\n'
```

可以使用通配符来遍历目录里面的文件。进行此操作的时候，你必须在文件名或者路径名中使用通配符。它会强制shell使用文件扩展匹配。如下：

```bash
for file in /home/chunxiao/test/*
do
	if [ -d "$file" ]; then
    	echo "is a dir"
    fi
done
```

注意这$file使用引号引用起来了，是因为文件名可以包含空格，如果不使用引号并且文件名包含了空格会产生错误：

```bash
./test: line6: [: too many arguments
```

#### C语言风格的for循环

基本格式如下：

```bash
for (( a = 1 ; a < 10; a++))
do
	echo $a
done
```

注意，有一些没有遵循标准的bash shell for命令：

- 给变量赋值可以有空格
- 条件中的变量不以美元开头
- 迭代过程的算式未用expr命令

C语言风格的for循环允许迭代多个变量。

```bash
for ((a=1, b=10; a < 10; a++, b--))
do
	echo "$a, $b"
done
```

#### while循环

while循环的基本格式如下:

```bash
while testCommand
do
	other command
done
```

while循环的test命令和if-then语句的定义是一样的。可以使用普通的bash shell命令也可以使用test命令作为条件。

常见用法如下:

```bash
a=10
while [ $a -gt 0 ]
do
	echo "$a"
    a=$[ $a - 1 ]
done
```

在while循环当中可以指定多个test命令。

```bash
a=3

while echo $a
	[ $a -ge 0 ]
do
	ehco "in loop"
    a=$[ $a - 1 ]
done

执行后输出：

3
in loop
2
in loop
1
in loop
0
in loop
-1
```

在多个命令的while循环当中，每次迭代的时候测试命令都会执行，注意每个测试命令都是在单独的一行上。

#### until命令

until命令和while命令工作的方式完全相反。until命令要求你指定一个通常输出非零退出状态码的测试命令。只有测试命令的退出状态码非零，bash shell才会执行do语句，当退出状态返回0，循环结束。

```bash
until testCommand
do
	commands
done
```

类似while循环，until命令语句中也可以有多个测试命令。

#### 嵌套循环

for循环的嵌套循环例子如下：

```bash
for (( a = 1; a <= 3; a++))
do
	for (( b=1; b <=3; b++ ))
    do
    	echo "b=$b"
    done
    echo "a=$a"
done
```

当然在while循环中也可以嵌套for循环。

#### 控制循环

有两种情况可以跳出循环：

- break命令
- continue命令

break命令可以退出任意类型的循环。

```bash
for i in 1 2 3 4 5; do
  if [ $i -eq 4 ]; then
	break
  fi
  echo "$i"
done
```

break可以跳出外部循环。break命令接受单个命令行参数值：

break n

默认n为1。看如下例子：

```bash
for ((i=0; i < 3 ; i++)); do
	echo "i = $i"
    for ((j = 0; j < 5; j++)); do
    	if [ $j -eq 1 ]; then
        	break 2
        fi
        echo "j = $j"
    done
done

# 输出为
i = 0
j = 0
```

continue命令的跳过当前循环体，但不终止循环。

```bash
for ((i=0; i < 3 ; i++)); do
        echo "i = $i"
    for ((j = 0; j < 5; j++)); do
        if [ $j -eq 1 ]  || [ $j -eq 2 ]; then
                continue
        fi
        echo "j = $j"
    done
done

#输出为
i = 0
j = 0
j = 3
j = 4
i = 1
j = 0
j = 3
j = 4
i = 2
j = 0
j = 3
j = 4
```

continue命令和break命令一样可以指定需要继续哪级循环。

#### 处理循环的输出

可以将for循环的输出到指定文件当中

```bash
for ((i=0; i < 3 ; i++)); do
        echo "i = $i"
    for ((j = 0; j < 5; j++)); do
        if [ $j -eq 1 ]  || [ $j -eq 2 ]; then
                continue
        fi
        echo "j = $j"
    done
done > a.txt
```

执行完上面命令后会将输出重定向到a.txt当中，而不是显示在屏幕上。

同样也可以将循环的结果管道符传给另外的命令。

```bash
for ((i=0; i < 3 ; i++)); do
        echo "i = $i"
    for ((j = 0; j < 5; j++)); do
        if [ $j -eq 1 ]  || [ $j -eq 2 ]; then
                continue
        fi
        echo "j = $j"
    done
done | sort
```

---EOF---
