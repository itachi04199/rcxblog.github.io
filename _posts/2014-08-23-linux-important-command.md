---
layout: post
title: linux重要命令
categories: linux
tags: linux
---

### linux重要命令

#### grep命令：

语法:grep [参数] pattern [文件名]

```
grep chunxiao /etc/passwd
grep -v chunxiao /etc/passwd #显示不包含chunxiao的行内容
grep -c chunxiao /etc/passwd #显示包含chunxiao的行数
grep -i chunxiao /etc/passwd #搜索时候忽略大小写
grep -r chunxiao /home/ #在home目录下查找，包括子目录
grep -rl chunxiao /home/ #在home目录下查找，包括子目录，只显示文件名
```

#### find命令：

语法：find 路径 约束条件

```
find /etc -name "*mail*" #在etc目录下查找文件名包含mail的文件。
find /etc -size +1M #查找etc目录下大于1M的文件
find /etc/ -type d  #查找etx目录下类型是目录的文件
find /etc/ -user root #查找etx目录下所有者为root的文件
find /etc/ -size +1M -exec ls -l {} \; #查找etc目录下大于1M的文件，并且用ls输出出来
#find ... -exec 命令 {} \;   {}代表find查询的结果。\代表转义符。;代表结果。
```

#### sort命令：

语法：sort [参数] 文件

```
sort sort.txt    #升序
sort -r sort.txt #降序
sort -t ' ' -k 2 sort #以空格当做分隔符，根据第2列升序排序
chunxiao@chunxiao-VirtualBox:~/chunxiao/$ cat sort 
a 7
c 2
d 3
v 1
e 6
f 12

chunxiao@chunxiao-VirtualBox:~/chunxiao/$ sort -t' ' -k 2 sort 
v 1
f 12
c 2
d 3
e 6
a 7

#可以看到12居然排到了第2位置，可以使用n参数来正确排序

chunxiao@chunxiao-VirtualBox:~/chunxiao/$ sort -t' ' -k 2n sort 
v 1
c 2
d 3
e 6
a 7
f 12
```

#### ls命令：

```
ls -a #显示所有文件，包括隐藏文件
ls -l #显示文件详细信息
ls -d #查看目录的信息
ls -i #查看文件的i节点，每个文件都有一个数字标识。
```

#### chmod命令：

对文件和目录权限的一些解释：

| 代表字符 | 权限 | 对文件的含义 | 对目录的含义 |
| :------------ | :------------ | :------------------ | :----------------- |
| r | 读权限 | 可以查看文件内容(cat、more、head、tail) | 可以列出目录中的内容(ls) |
| w | 写权限 | 可以修改文件内容(echo、vi) | 可以在目录中创建、删除文件(touch、mkdir、rm) |
| x | 执行权限 | 可以执行文件(命令、脚本) | 可以进入目录 |

```
#chmod [u、g、o]  [+、-、=]  [r、w、x]，例子如下：
chmod u+wx     chmod g-rx  chmod o=rwx 
#r-4、w-2、x-1
chmod  755 a   #相当于给a文件赋予权限为  rwxr-xr-x
```

#### uniq命令：

如果文件中有多行完全相同的内容，可以使用uniq来删除重复的行。

```
uniq a.txt 
uniq -i a.txt #忽略大小写
uniq -c a.txt #计算重复的行数
sort -t : -k 2n sort | uniq #一般会与sort一起使用，排序完成后删除重复的行

```

后续的命令，随着学习在进行补充，还有这里的命令随着深入的学习后续也会补充其他用法。

本篇完

END

