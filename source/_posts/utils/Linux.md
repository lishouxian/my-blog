---
title: 'Linux级'
date: 2020-10-24 21:13:45
tags: [学习]
categories: utils
published: true
hideInList: false
feature: 
isTop: false
---
### Linux的学习步骤

第 1 阶段：linux 环境下的基本操作命令，包括 文件操作命令(rm mkdir chmod, chown) 编辑工具使用（vi vim）linux 用户管理(useradd userdel usermod)等

第 2 阶段：linux 的各种配置（环境变量配置，网络配置，服务配置）

第 3 阶段：linux 下如何搭建对应语言的开发环境（大数据，JavaEE, Python 等） 第 4 阶段：能编写 shell 脚本，对 Linux 服务器进行维护。

第 5 阶段：能进行安全设置，防止攻击，保障服务器正常运行，能对系统调优。

第 6 阶段：深入理解 Linux 系统（对内核有研究），熟练掌握大型网站应用架构组成、并熟悉各个环节的部署和维护方法。

<!-- more -->

##  Linux基础

### 文件基本属性

Linux 系统是一种典型的多用户系统，不同的用户处于不同的地位，拥有不同的权限。

为了保护系统的安全性，Linux 系统对不同的用户访问同一文件（包括目录文件）的权限做了不同的规定。

在 Linux 中我们通常使用以下两个命令来修改文件或目录的所属用户与权限：

- chown (change ownerp) ： 修改所属用户与组。
- chmod (change mode) ： 修改用户的权限。

```shell
[root@www /]# ls -l
total 64
dr-xr-xr-x   2 root root 4096 Dec 14  2012 bin
dr-xr-xr-x   4 root root 4096 Apr 19  2012 boot
……
```

![img](https://tva1.sinaimg.cn/large/007S8ZIlly1giv3yh00s4j30e80b8wfu.jpg)

### Linux的文件与目录管理

linux 的文件系统是采用级层式的树状目录结构，在此结构中的最上层是根目录“/”，然后在此目录下再创建其他的目录。

深刻理解 linux 树状文件目录是非常重要的，这里我给大家说明一下。

```shell
[root@aliC ~]# cd /
[root@aliC /]# ls
bin   code  etc   lib    media  opt   root  sbin  sys  usr
boot  dev   home  lib64  mnt    proc  run   srv   tmp  var
```

记住一句经典的话：在 Linux 世界里，一切皆文件。

1)  linux 的目录中有且只要一个根目录 `/`

2)  linux 的各个目录存放的内容是规划好，不用乱放文件。

3)  linux 是以文件的形式管理我们的设备，因此 linux 系统，一切皆为文件。

- **绝对路径：**
  路径的写法，由根目录 **/** 写起，例如： /usr/share/doc 这个目录。
- **相对路径：**
  路径的写法，不是由 **/** 写起，例如由 /usr/share/doc 要到 /usr/share/man 底下时，可以写成： **cd ../man** 这就是相对路径的写法。

### 处理目录的常用命令

接下来我们就来看几个常见的处理目录的命令吧：

- ls（英文全拼：list files）: 列出目录及文件名
- cd（英文全拼：change directory）：切换目录
- pwd（英文全拼：print work directory）：显示目前的目录
- mkdir（英文全拼：make directory）：创建一个新的目录
- rmdir（英文全拼：remove directory）：删除一个空的目录
- cp（英文全拼：copy file）: 复制文件或目录
- rm（英文全拼：remove）: 移除文件或目录
- mv（英文全拼：move file）: 移动文件与目录，或修改文件与目录的名称

### vi和vim的使用

vim

## crond 任务调度

TODO



## 文件系统

分区与文件系统：

对分区进行格式化是为了在分区上建立文件系统。一个分区通常只能格式化为一个文件系统，但是磁盘整列等技术可以使一个分区格式化为多个文件系统。

组成：

最重要的几个组成部分

- inode：一个文件占用一个iNode，记录文件的属性，同时记录此文件的内容的block编号；
- block：记录文件的内容，文件太大时，会占用多个block

出此之外还包括：

- superblock：记录文件系统的整体性喜，包括iNode和block的总量，使用量、剩余量，以及文件系统的格式与相关信息等
- block bitmap：记录block是否被使用的位图

### 文件系统

对于Ext2 系统，当你要读取一个文件的内容的时候，现在inode中查找文件内容所在的所有block，然后按照顺序将所有block的内容读出来

对于FAT文件系统，他没有inode，每个block中存储着下一个block的编号。

磁盘碎片

只一个文件内容所在的block过于分散，导致磁盘磁头移动距离过大从而降低磁盘读写的性能。

### block

在 Ext2 文件系统中所支持的 block 大小有 1K，2K 及 4K 三种，不同的大小限制了单个文件和文件系统的最大大小。

| 大小         | 1KB  | 2KB   | 4KB  |
| ------------ | ---- | ----- | ---- |
| 最大单一文件 | 16GB | 256GB | 2TB  |
| 最大文件系统 | 2TB  | 8TB   | 16TB |

一个 block 只能被一个文件所使用，未使用的部分直接浪费了。因此如果需要存储大量的小文件，那么最好选用比较小的 block。

### inode

inode 具体包含以下信息：

- 权限 (read/write/excute)；
- 拥有者与群组 (owner/group)；
- 容量；
- 建立或状态改变的时间 (ctime)；
- 最近读取时间 (atime)；
- 最近修改时间 (mtime)；
- 定义文件特性的旗标 (flag)，如 SetUID...；
- 该文件真正内容的指向 (pointer)。

inode 具有以下特点：

- 每个 inode 大小均固定为 128 bytes (新的 ext4 与 xfs 可设定到 256 bytes)；
- 每个文件都仅会占用一个 inode。

inode 中记录了文件内容所在的 block 编号，但是每个 block 非常小，一个大文件随便都需要几十万的 block。而一个 inode 大小有限，无法直接引用这么多 block 编号。因此引入了间接、双间接、三间接引用。间接引用让 inode 记录的引用 block 块记录引用信息。

### 目录

建立一个目录时，会分配一个inode与至少一个block。block记录的内容是目录下所有文件的inode编号以及文件名。

可以看到文件的inode本身不记录文件名，文件名记录在目录中，因此新增文件，删除文件，更改文件名这些操作与目录的写权限有关。

### 日志

如果突然断电，那么文件系统会发生错误，例如断电前只修改了 block bitmap，而还没有将数据真正写入 block 中。

ext3/ext4 文件系统引入了日志功能，可以利用日志来修复文件系统。

### 挂载

挂载利用目录作为文件系统的进入点，也就是说，进入目录之后就可以读取文件系统的数据。

### 目录配置

为了使不同 Linux 发行版本的目录结构保持一致性，Filesystem Hierarchy Standard (FHS) 规定了 Linux 的目录结构。最基础的三个目录如下：

- / (root, 根目录)
- /usr (unix software resource)：所有系统默认软件都会安装到这个目录；
- /var (variable)：存放系统或程序运行过程中的数据文件。

![img](https://tva1.sinaimg.cn/large/007S8ZIlly1giy2ngzqb2j30ic08p0to.jpg)

## 进程管理

### 查看进程

#### 1.ps 查看进程

查看自己的进程

```shell
[root@aliC ~]# ps -l
F S  UID   PID  PPID C PRI NI ADDR SZ WCHAN TTY     TIME CMD
0 S   0 168664 168663 0 80  0 - 7385 -   pts/0  00:00:00 bash
0 R   0 168687 168664 0 80  0 - 11404 -   pts/0  00:00:00 ps
```

注意：

`PID`进程号

`PPID`父进程

查看系统所有的进程`ps aux`

查看特定的进程`ps aux | grep threadx` `ps -aux | grep python`

以全格式显示进程 

#### 2. pstree 查看进程树

查看所有进程树

```shell
pstree -A
```

### 进程状态

![进程管理](https://tva1.sinaimg.cn/large/007S8ZIlly1giv2slez5yj31ae0u043k.jpg)

| 状态 | 说明                                                         |
| ---- | ------------------------------------------------------------ |
| R    | running or runnable (on run queue) 正在执行或者可执行，此时进程位于执行队列中。 |
| D    | uninterruptible sleep (usually I/O) 不可中断阻塞，通常为 IO 阻塞。 |
| S    | interruptible sleep (waiting for an event to complete) 可中断阻塞，此时进程正在等待某个事件完成。 |
| Z    | zombie (terminated but not reaped by its parent) 僵死，进程已经终止但是尚未被其父进程获取信息。 |
| T    | stopped (either by a job control signal or because it is being traced) 结束，进程既可以被作业控制信号结束，也可能是正在被追踪。 |

#### SIGCHLD

当一个子进程改变了它的状态时（停止运行，继续运行或者退出），有两件事会发生在父进程中：

- 得到 SIGCHLD 信号；
- waitpid() 或者 wait() 调用会返回。

其中子进程发送的 SIGCHLD 信号包含了子进程的信息，比如进程 ID、进程状态、进程使用 CPU 的时间等。

在子进程退出时，它的进程描述符不会立即释放，这是为了让父进程得到子进程信息，父进程通过 wait() 和 waitpid() 来获得一个已经退出的子进程的信息。

#### wait()

```c
pid_t wait(int *status)Copy to clipboardErrorCopied
```

父进程调用 wait() 会一直阻塞，直到收到一个子进程退出的 SIGCHLD 信号，之后 wait() 函数会销毁子进程并返回。

如果成功，返回被收集的子进程的进程 ID；如果调用进程没有子进程，调用就会失败，此时返回 -1，同时 errno 被置为 ECHILD。

参数 status 用来保存被收集的子进程退出时的一些状态，如果对这个子进程是如何死掉的毫不在意，只想把这个子进程消灭掉，可以设置这个参数为 NULL。

#### waitpid()

```c
pid_t waitpid(pid_t pid, int *status, int options)Copy to clipboardErrorCopied
```

作用和 wait() 完全相同，但是多了两个可由用户控制的参数 pid 和 options。

pid 参数指示一个子进程的 ID，表示只关心这个子进程退出的 SIGCHLD 信号。如果 pid=-1 时，那么和 wait() 作用相同，都是关心所有子进程退出的 SIGCHLD 信号。

options 参数主要有 WNOHANG 和 WUNTRACED 两个选项，WNOHANG 可以使 waitpid() 调用变成非阻塞的，也就是说它会立即返回，父进程可以继续执行其它任务。

### 孤儿进程

一个父进程退出，而它的一个或多个子进程还在运行，那么这些子进程将成为孤儿进程。

孤儿进程将被 init 进程（进程号为 1）所收养，并由 init 进程对它们完成状态收集工作。

由于孤儿进程会被 init 进程收养，所以孤儿进程不会对系统造成危害。

### 僵尸进程

一个子进程的进程描述符在子进程退出时不会释放，只有当父进程通过 wait() 或 waitpid() 获取了子进程信息后才会释放。如果子进程退出，而父进程并没有调用 wait() 或 waitpid()，那么子进程的进程描述符仍然保存在系统中，这种进程称之为僵尸进程。

僵尸进程通过 ps 命令显示出来的状态为 Z（zombie）。

系统所能使用的进程号是有限的，如果产生大量僵尸进程，将因为没有可用的进程号而导致系统不能产生新的进程。

要消灭系统中大量的僵尸进程，只需要将其父进程杀死，此时僵尸进程就会变成孤儿进程，从而被 init 进程所收养，这样 init 进程就会释放所有的僵尸进程所占有的资源，从而结束僵尸进程。



































