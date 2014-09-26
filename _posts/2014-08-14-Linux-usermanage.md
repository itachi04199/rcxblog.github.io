---
layout: post
title: linux 用户管理
categories: linux 学习笔记
tags: linux
---

### Linux用户管理

GID,UID。不同的用户有不同的UID(user id)，不同的组有不用的GID(group id)。

Linux系统的用户分为三类，普通用户、root用户和系统用户。

- 普通用户通常是只所有使用Linux系统的真实用户，这类用户可以使用用户名和密码登录系统。普通用户的UID一般大于500。
- root用户的UID是0。可以对系统完全控制，使用root用户的时候需要小心。
- 系统用户指系统运行时必须有的用户，但不是真实的使用者。例如tomcat、mysql等用户。系统用户的UID在1-499之间。

Linux系统中还有用户组的概念。基本上与用户的概念相似。

```
id      #查看自己UID
groups  #查看自己所属的用户组
```

Linux中的/etc/passwd和/etc/shadow 文件。在登录Linux系统需要用户名和密码。用户名和密码的信息就记录在这2个文件当中。

```
chunxiao@chunxiao-VirtualBox:~$ cat /etc/passwd  #查看/etc/passwd文件里面的内容
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/bin/sh
bin:x:2:2:bin:/bin:/bin/sh
sys:x:3:3:sys:/dev:/bin/sh
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/bin/sh
man:x:6:12:man:/var/cache/man:/bin/sh
```

可以看到虽然每行内容不一样，但是格式相同。以:为分隔符分成7列，每一列的含义如下表：

|列数  | 含义  | 说明 | 
| :----- | :-------- | :---- |
|1 | 用户名 | UID的字符串表示形式，方便阅读|
|2 | 密码 | 在旧的UNIX系统当中，该字段是加密后的用户密码。现在都将密码存放在/etc/shadow文件中，这个地方都存为x。|
|3 | UID | UID|
|4 | GID | GID|
|5 | 说明栏 | 类似于“注释”，现在不使用|
|6 | 家目录 | 用户登录后，所处的目录|
|7 | 目录 | 用户登录后，所使用的|

看到上面的第二列存为x，是因为每个用户都有读取这个文件的权限，所以经过加密的密码也很容易被破解。Linux的做法是，将密码相关的信息保存到/etc/shadow文件中，默认只有root用户可以读取这个文件。

```
chunxiao@chunxiao-VirtualBox:~$ sudo cat /etc/shadow
root:!:16279:0:99999:7:::
daemon:*:15937:0:99999:7:::
bin:*:15937:0:99999:7:::
```

格式与/etc/passwd差不多，只不过这是分割成9列，每列的含义如下表：

|列数  | 含义  | 说明 | 
| :----- | :-------- | :---- |
|1 | 用户名 | UID的字符串表示形式，方便阅读|
|2 | 密码 | 加密后的密码 |
|3 | 密码的最近修改日 | 这个数字是1970-1-1到修改密码日的天数|
|4 | 密码不可修改的天数 | 修改密码后几天不可以再次修改密码，如果是0，则随时可以修改|
|5 | 密码重新修改的天数 | 考虑密码会泄露，设置一个修改时间，密码到期之前系统会提示用户修改密码|
|6 | 密码失效前提前警告的天数 | 设置提前几天提醒修改密码|
|7 | 密码失效宽限天数 | 如果密码过期，几天后失效，不可登录系统|
|8 | 帐号失效日期 | 一般为空|
|9 | 保留字段 | 保留字段|

添加用户可以使用下面的命令：

```
useradd tom #添加用户
useradd -u 555 lucy #指定UID来创建用户，UID必须不冲突
useradd -g tom tony #指定GID来创建用户
useradd -d /home/mydir user #指定家目录来创建用户
```

一般添加个新用户的过程是，首先在/etc/passwd和/etc/shadow末尾追加一条记录，同时分配给用户一个UID。接着为该用户创建家目录。比如上面的例子的目录是/home/tom，然后将/etc/skel下所有文件复制到/home/tom目录下面。最后创建一个跟用户名相同的用户组，并且这个用户属于这个用户组。

修改密码：

```
passwd tom  #root用户修改tom的密码
passwd      #普通用户修改自己的密码，普通用户后面不可以加参数
```

新创建的用户在不设置密码的时候是不可以登录系统的，查看/etc/shadow文件新创建的用户的一行如下：

```
user:!!:16295:0:99999:7:::
```

第二列是个'!!'号，说明该用户不可以登录系统，需要设置密码才可以。

修改用户：usermod，使用usermod命令其实就是对/etc/passwd和/etc/shadow文件进行修改操作。

```
usermod -d /home/alice_bew -m alice #修改alice用户的家目录 -m参数代表如果新的目录不存在，将自动创建。
usermod -L alice #将用户冻结，alice:!:16295:0:99999:7::: 查看/etc/shadow 文件，第2列密码前面出现!。
usermod -U alice #将用户解冻
```

删除用户：userdel，在删除用户时候，默认不会删除用户的家目录和邮件信息，可以使用-r参数来删除这些信息。

```
userdel alice #将alice用户删除，同时在/etc/passwd和/etc/shadow文件的记录也会被删除。
```

新增加用户组：groupadd,在linux当中使用/etc/group文件来记录用户组信息。

```
groupadd group1  #查看/etc/group 文件最后一行  group1:x:1002: ，新添加的组的GID为1002
```

删除用户组：groupdel,如果已有用户属于这个试图被删掉的组当中，该操作会失败。

```
groupdel group1
```

检查用户信息：users、who、w

```
users  #结果如下
chunxiao chunxiao  

who    #结果如下，第一列用户名，第二列终端，第三列登录时间
chunxiao tty7         2014-08-14 08:44
chunxiao pts/2        2014-08-14 08:44 (:0.0)

w      #结果如下，
 09:13:15 up 32 min,  2 users,  load average: 0.00, 0.01, 0.05
USER     TTY      FROM              LOGIN@   IDLE   JCPU   PCPU WHAT
chunxiao tty7                      08:44     ?     3.67s  0.17s gnome-session --session=ubuntu
chunxiao pts/2    :0.0             08:44    3.00s  0.28s  0.00s w
```

w命令显示8列，每一列的说明如下表：

|列号 | 说明|
| :------- | :------ |
|1 | 用户名|
|2 | 终端|
|3 | 如果用户从网络登录，则显示用户的远程主机的ip或者主机名|
|4 | 登录时间|
|5 | 用户闲置时间|
|6 | 与终端相关的当前所有运行进程消耗的CPU时间总量|
|7 | 当前what列所对应的进程小号的CPU时间总量|
|8 | 用户当前运行的进程|

调查用户：finger

```
chunxiao@chunxiao-VirtualBox:~$ finger #不加参数的形式
Login     Name       Tty      Idle  Login Time   Office     Office Phone
chunxiao  chunxiao   tty7           Aug 14 08:44
chunxiao  chunxiao   pts/2          Aug 14 08:44 (:0.0)

chunxiao@chunxiao-VirtualBox:~$ finger chunxiao #添加用户参数，显示详细的信息
Login: chunxiao       			Name: chunxiao
Directory: /home/chunxiao           	Shell: /bin/bash
On since Thu Aug 14 08:44 (CST) on tty7
On since Thu Aug 14 08:44 (CST) on pts/2 from :0.0
   2 seconds idle
No mail.  #邮件信息
No Plan.  #计划信息
```

切换用户:su、sudo。su命令是切换用户的意思，默认切换到root用户，需要输入root用户的密码。su命令可以添加‘-’参数，表示切换到root后使root的用户环境。就是/etc/passwd下面的家目录和使用的。

```
su   #默认切换到root
su - #默认切换到root，并且使用root的用户环境
su tom #切换到tom用户，需要知道tom的密码。
```

使用su命令是很方便的，但是需要知道切换用户的密码，如果很多人知道root的密码会不安全，所以我们可以使用sudo这个命令。这个命令的使用方式是sudo加上需要执行的命令。一般人没权限看/etc/shadow文件，可以使用sudo命令，如下：

```
sudo cat /etc/shadow
```

---EOF---
