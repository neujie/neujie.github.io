---
layout: post
title: 如何手动释放Linux内存
categories:
- Technology
tags:
- 系统
---
[转载:手工释放linux内存——/proc/sys/vm/drop_caches](https://www.cnblogs.com/jackhub/p/3736877.html)

--手工释放linux内存——/proc/sys/vm/drop_caches

总有很多朋友对于Linux的内存管理有疑问，之前一篇日志似乎也没能清除大家的疑虑。而在新版核心中，似乎对这个问题提供了新的解决方法，特转出来给大家参考一下。最后，还附上我对这方法的意见，欢迎各位一同讨论。
    当在Linux下频繁存取文件后，物理内存会很快被用光，当程序结束后，内存不会被正常释放，而是一直作为caching。这个问题，貌似有不少人在问，不过都没有看到有什么很好解决的办法。那么我来谈谈这个问题。

# 一、通常情况 先来说说free命令:

    [root@server ~]# free -m
    total　　used　　free　　shared　　buffers　　cached
    Mem:  　　 249　　 163 　　 86　　　　 0 　　　　10 　　　　94
    -/+ buffers/cache:    58 　　 191
    Swap: 　　511 　　　0　　　 511

其中:

* total 内存总数
* used 已经使用的内存数
* free 空闲的内存数
* shared 多个进程共享的内存总额。

buffers Buffer Cache和cached Page Cache 磁盘缓存的大小

* -buffers/cache 的内存数:used - buffers - cached
* +buffers/cache 的内存数:free + buffers + cached

可用的memory=free memory+buffers+cached。

有了这个基础后，可以得知，我现在used为163MB，free为86MB，buffer和cached分别为10MB，94MB。 那么我们来看看,如果我执行复制文件,内存会发生什么变化.

    [root@server ~]# cp -r /etc ~/test/
    [root@server ~]# free -m
    total used free shared buffers cached
    Mem: 249 244 4 0 8 174
    -/+ buffers/cache: 62 187
    Swap: 511 0 511

在我命令执行结束后，used为244MB，free为4MB，buffers为8MB，cached为174MB，天呐，都被cached吃掉了。别紧张，这是为了提高文件读取效率的做法。

为了提高磁盘存取效率，Linux做了一些精心的设计，除了对dentry进行缓存（用于VFS，加速文件路径名到inode的转换），还采取了两种主要Cache方式:Buffer Cache

和Page Cache。前者针对磁盘块的读写，后者针对文件inode的读写。这些Cache有效缩短了 I/O系统调用（比如read，write，getdents）的时间。

那么有人说过段时间，linux会自动释放掉所用的内存。等待一段时间后，我们使用free再来试试，看看是否有释放？

    [root@server test]# free -m
    total used free shared buffers cached
    Mem: 249 244 5 0 8 174
    -/+ buffers/cache: 61 188
    Swap: 511 0 511

似乎没有任何变化。（实际情况下，内存的管理还与Swap有关）

那么我能否手动释放掉这些内存呢？回答是可以的！

# 二、手动释放缓存

**/proc是一个虚拟文件系统，我们可以通过对它的读写操作做为与kernel实体间进行通信的一种手段。也就是说可以通过修改/proc中的文件，来对当前**

kernel的行为做出调整。那么我们可以通过调整/proc/sys/vm/drop_caches来释放内存。操作如下:

首先，/proc/sys/vm/drop_caches的值，默认为0。

    [root@server test]# cat /proc/sys/vm/drop_caches 0

手动执行sync命令（描述:sync 命令运行 sync 子例程。如果必须停止系统，则运行sync 命令以确保文件系统的完整性。sync 命令将所有未写的系统缓冲区写到磁盘中，包含已修改的 i-node、已延迟的块 I/O 和读写映射文件）

    [root@server test]# sync

将/proc/sys/vm/drop_caches值设为3

    [root@server test]# echo 3 > /proc/sys/vm/drop_caches

    [root@server test]# cat /proc/sys/vm/drop_caches 3

    [root@server test]# free -m total used free shared buffers cached
    Mem: 249 66 182 0 0 11
    -/+ buffers/cache: 55 194
    Swap: 511 0 511

再来运行free命令，会发现现在的used为66MB，free为182MB，buffers为0MB，cached为11MB。那么有效的释放了buffer和cache。

# 有关/proc/sys/vm/drop_caches的用法在下面进行了说明

    /proc/sys/vm/drop_caches (since Linux 2.6.16) Writing to this file causes the kernel to drop clean caches, dentries and inodes from memory, causing that memory to become free.

    To free pagecache, use echo 1 > /proc/sys/vm/drop_caches;
    to free dentries and inodes, use echo 2 > /proc/sys/vm/drop_caches;
    to free pagecache, dentries and inodes, use echo 3 > /proc/sys/vm/drop_caches.

    Because this is a non-destructive operation and dirty objects are not freeable, the user should run sync first.

