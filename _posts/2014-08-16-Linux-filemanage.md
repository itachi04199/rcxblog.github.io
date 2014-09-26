---
layout: post
title: linux 文件管理
categories: linux 学习笔记
tags: linux 基础 笔记
---
### Linux文件管理

在Linux系统当中，所有文件和目录都存放在根目录下，即/下。FHS(文件系统层次标准)定义了在根目录下的主要目录以及每个目录存放什么文件。如下表：

|目录|目录用途|
|:---|:----|
|/bin|常见的用户指令|
|/boot|内核和启动文件|
|/dev|设备文件|
|/etc|系统和服务的配置文件|
|/home|系统默认的普通用户的家目录|
|/lib|系统函数库目录|
|/lost+found|ext3文件系统需要的目录，用于磁盘检查|
|/mnt|系统加载文件系统时常用的挂载点|
|/opt|第三方软件安装目录|
|/proc|虚拟文件系统|
|/root|root用户的家目录|
|/sbin|存放系统管理命令|
|/tmp|存放临时文件目录|
|/usr|存放与用户直接相关的文件和目录|
|/media|系统用来挂载光驱等临时文件系统的挂载点|

在每个目录下 ，都会存放两个特殊的目录，.和..这2个目录。.代表的是当前目录，..代表上级目录。

使用touch命令创建文件：

```
touch test.txt  #创建名为test.txt的文件
```

使用touch创建文件的时候，如果该文件在这个目录下存在了，那么这个命令不会对当前的同名文件造成影响，因为它不会修改文件的内容，实际上只会更新文件的创建时间属性。

删除文件：rm

```
rm test.txt
```

移动或重命名文件：mv

```
mv test.txt /test/   #移动文件到/test 目录下
mv test.txt test.md  #将文件改名为test.md
mv test.txt /test/test.md #移动文件并且改名
```

查看文件：cat,该命令是concatenate的简写。

```
cat test.txt    #查看文件内容
cat -n test.txt #-n 参数可以显示每行的行号
```

查看文件头：head,有时候文件非常大，cat查看出来的内容太多，我们只想看文件的开头部分。head默认显示前10行内容。

```
head test.txt       #显示文件前10行
head -n 20 test.txt #显示文件前20行
```

查看文件尾部：tail，基本与head相似只是查看文件的后面行数,默认显示最后10行，也可以使用-n参数来指定。

```
tail test.txt
tail -n 20 test.txt
tail -f test.txt  #使用-f参数可以动态查看这个文件，用于查看日志。
```

进入目录：cd（change directory）

```
cd /tmp   #进入tmp目录
```

创建目录：mkdir

```
mkdir dir1   #创建dir1目录
mkdir -p dir2/dir3/dir4 #一次性创建多级目录
```

删除目录：rmdir和rm,rmdir这个命令是可以删除空目录。(r  recursive，递归。)所以一般对文件夹进行操作的都会加上-r参数。

```
rmdir dir3
rm -r dir1  #删除dir1目录下的所有文件
rm -rf dir1 #删除dir1目录，并且不需要确认
```

文件和目录的复制：cp

```
cp a b #复制a，新文件名称是b
cp a /dir/b #将a文件复制到dir目录下，文件名叫b
cp a /dir/  #将a文件复制到dir目录下，文件名叫a
cp -r dir dir2 #复制文件夹，-r参数
```

查看文件或目录的权限：

```
ls -al

总用量 24
drwxrwxr-x  3 chunxiao chunxiao 4096  8月 16 13:20 .
drwxrwxr-x 11 chunxiao chunxiao 4096  8月 10 15:06 ..
drwxrwxr-x  4 chunxiao chunxiao 4096  8月 16 13:22 dir
-rwxrwxr-x  1 chunxiao chunxiao   25  8月 10 15:12 sh1.sh
-rw-rw-r--  1 chunxiao chunxiao  113  8月 16 13:05 y
-rw-rw-r--  1 chunxiao chunxiao  113  8月 16 13:20 y1

```

上面详细的输出了文件信息。每个文件都有7列。第一列是文件类别和权限。文件类别如下表：

|第一个字符可能的值|含义|
|:---|:---|
|d|目录|
|-|普通文件|
|l|链接文件|
|b|块文件|
|c|字符文件|
|s|socket|
|p|管道文件|

接下来的每3个字符为1组，第2-4个字符代表该文件所有者(user)权限，第5-7字符代表该文件所属组(group)权限，第8-10字符代表其他用户(others)权限。每组都是rwx来表示，有权限就显示，无权限显示-。

第二列代表连接数目，第三列代表文件所有者，第四列代表文件是所有组，第五列文件大小，第六列是文件的创建时间或者最近修改时间，第七列是文件名。

文件还有一些隐藏属性，必须用lsattr来显示：

```
lsattr a.txt  #显示结果如下
-------------e- a.txt
```

如果需要设置隐藏属性需要使用chattr命令。可以给文件增加a属性，a属性即使是root用户也不可以删除这个文件，只能在文件尾部追加。

```
chattr +a a.txt
```

改变文件权限：chmod,linux下的每一个文件都定义了所有者(user),所属组(group),其他人(others)，我们可以使用u、g、o来代表他们。增加权限使用+，减少权限用-，详细权限用=。请看下面的例子：

```
chmod u+r a.txt
chmod u-r a.txt
chmod u=rw- a.txt
```

上面只是对u进行操作权限，其他的跟这个类似，这个方式很麻烦，如果想同时想对u、g、o进行操作需要操作3次。所以，我们定义r=4,w=2,x=1，如果权限是rwx，则数字表示为7，如果权限是r-x，则数字表示为5。那么就可以使用下面的方式来改变权限。

```
chmod 754 a.txt
chmod -R 754 dir  #给目录授权，使用-R参数，这个目录下面所有文件和子目录都是754这个权限。
```

改变文件的所有者：chown，同时还具备修改所属组的功能。

```
chown john a.txt  #将a.txt的所有者改成john
chown :john a.txt #将a.txt的所属组改成john
chown -R john dir #修改目录的所有者为john
chown -R john:john dir #同时对目录修改所有者和所属组
```

还可以使用chgrp来改变所属组：

```
chgrp tom a.txt
chgrp -R tom dir
```

默认权限和umask，当新创建个文件或者目录的时候这个文件和目录会有默认的权限，这个权限是如何而来，首先看下默认的权限是什么：

```
touch y
ls -l y  #输出如下
-rw-rw-r-- 1 chunxiao chunxiao 0  8月 16 15:29 y

mkdir d
ls -dl d #输出如下
drwxrwxr-x  2 chunxiao chunxiao 4096  8月 16 15:31 d
```

注意到文件的默认权限是664，目录的默认权限是775。在/etc/login.defs文件中看到了UMASK被设置成002。
在Linux下，创建文件的默认权限是666，(因为创建的文件不能具有执行权限)。创建目录的默认权限是777。所以可以明显的看到umask的作用了，当创建文件的时候默认权限666-002=664，创建目录的时候默认权限777-002=775。

file命令查看文件类型，前面已经讲到了ls命令输入的几种文件类型。

```
file a.txt  #查看文件
file /tmp   #查看目录
```

一般查找：find，详细用法参见man find

```
find PATH -name FileName
find / -name passwd   在/目录下查找文件名为passwd的文件
```

数据库查找：locate，与find不同，locate依赖一个数据库文件，linux系统默认每天会检索一下系统当中的所以文件，然后将检索到的文件记录到数据库当中。运行locate命令的时候可以直接到数据库当中查找，所以使用locate更快些。在执行locate之前一般需要执行updatedb，来更新数据库。

查找执行文件：which/whereis

```
which passwd   #输出如下
/usr/bin/passwd

whereis passwd #输出如下，可以查找到其二进制文件也可以查找到man文件
passwd: /usr/bin/passwd /etc/passwd /usr/bin/X11/passwd /usr/share/man/man5/passwd.5.gz /usr/share/man/man1/passwd.1ssl.gz /usr/share/man/man1/passwd.1.gz

```

文件压缩和打包，gzip/gunzip用来压缩和解压缩单个文件的工具。

```
gzip install.log   #gzip后原文件变成压缩的
gunzip install.log.gz #gunzip后压缩文件变成原文件
```

tar命令可以打包文件，还可以把整个目录整合成一个包，整合包的同时还可以同时使用gzip的功能进行压缩。

```
tar -zcvf dir.tar.gz dir/  #-z是代表使用gzip，-c是创建压缩文件，-v是显示当前被压缩的文件，-f是指使用文件名。
```

本篇完
END

