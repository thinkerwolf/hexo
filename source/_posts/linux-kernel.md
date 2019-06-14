---
title: Linux系列-内核组成
date: 2017-08-13 10:07:16
categories:
- linux
tags:
- linux
---

Linux内核的主要工作
- 系统内存管理
- 软件程序管理
- 硬件管理
- 文件系统管理

#### 系统内存管理
内核管理真实的“物理内存”和“虚拟内存”，有一块真是内存的区域“交换区”，用来将虚拟内存中的地址内容来回传递到真实的物理内存中。

内存地址被组织成块，称作页“pages”。Linux内核会定位物理内存和交换区的“页”。内核中包含一张“页”数据表，表明哪些页在硬盘中，哪些页在交换区中。

内核会实时监控哪些“页”在使用，哪些页在空闲中。如果某些“页”在长时间未使用，内核会把这些“页”交换出物理空间中，放入交换区，如果某个程序访问被交换出的“页”，需要在物理内存中重新开辟一块新的空间，随后将“页”重新复制到物理内存中。

**cat /proc/meminfo**命令可以查看当前内存的状态。

Linux内核中的进程会有私有的内存，出于安全性考虑。其他进程无法访问到私有内存。为了加强数据分享，用户可以创建共享内存。共享内存中会存储创建内存段的用户，时间、权限等等。

{% asset_img linux_kernel.png This is an example image %}


#### 软件程序管理
Linux内核将运行的程序称作一个进程，Linux内核在启动时会先启动一个 init process(初始化进程)，内核会为次进程在虚拟内存中开辟一块特殊的区域用来存储数据和代码，随后内核会启动附带的进程。

Linux内核有多种启动等级（Level），通常是Level1，Level3，Level5，每一种启动等级都会启动不同数量的进程。等级越高，启动的进程数量越多。从系统基本进程 -> 图形界面进行。

#### 硬件管理
Linux内核中把一个硬件抽象成一个文件，Linux中有三种硬件文件。
- Character
- Block
- NetWork

Character表示一次只能处理一个字符的硬件。
Block表示可以处理一块数据的硬件，比如硬盘。
Network表示可以发送和接收数据的硬件，比如网卡。

Linux为系统的硬件创建特殊的文件，名叫节点（Node）。系统与硬件交流就是通过节点来完成的。每个Node都使用一对数字(major num，monitor num)来标识，相似的设备用相同的major num来表示。在/dev文件夹中可以看到所有硬件Node文件。

下面的图片是Node文件的详细信息。第一列表示读写权限，读写权限第一个字母表示文件类型。b(Block)、c(Character)。

{% asset_img linux_node.png This is an example image %}

#### 文件系统管理
{% asset_img linux_filesystem.png This is an example image %}


## The GNU UNITIES
GNU核心工具有三种。
- 处理文件工具
- 操作文件工具
- 管理进程工具

#### The Shell
shell是一种特殊的交互式工具，用户可以利用Shell查看、修改、删除文件、关机等一系列操作。可以将Shell脚本写入文件中，Linux内核会读取Shell文件并执行。Linux支持几种Shell脚本，其中最常用的是Bash Shell。