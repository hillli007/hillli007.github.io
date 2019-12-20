---
title: lfs -- 从源码编译系统
date: 2018-10-15
tags:
---

### 引言

为什么要准备

### 准备工具

- 下载BOOK和源码
- 检查宿主系统工具与库是否符合要求

### 准备磁盘


### 设置环境

源码和工具链目录要先chown给lfs用户

~~~
chown -v lfs $LFS/tools
chown -vR lfs $LFS/sources
~~~

su - lfs 时其实是非登录Shell，因为.bash_profile直接执行的bash，

~~~
env -i NAME1=VALUE1 NAME2=VALUE2 <command-line>
~~~

使用指定的环境变量执行命令行<command-line>

~~~
cat > ~/.bash_profile << "EOF"
exec env -i HOME=$HOME TERM=$TERM PS1='\u:\w\$ ' /bin/bash
EOF
~~~

### 创建工具链

创建lfs用户来操作，避免root权限太大破坏宿主系统

$LFS/tools目录下构建一个小型linux系统

- 用宿主系统的编译工具创建临时工具链"$LFS/tools -> /tools"，目的是为了用来编译LFS，为了保持LFS不被宿主系统污染。
- 工具链有基本的binutils、glic、gcc、linux api header等，但是会加上前缀用于与宿主系统工具区分开来。
- 工具链里的工具不是LFS真正使用的，后续编译系统时需要再次编译binutils、glic、gcc、linux api header等。

### 安装基本系统软件

lfs还没有能够引导起来，但是又必须提前安装好必要的系统软件，这一步就需要宿主系统来帮助完成。

但是这里有两个问题

准备虚拟内核文件系统

chroot命令用来在指定的根目录下运行指令

~~~
chroot /mnt/lfs /tools/bin/env -i \
    HOME=/root                  \
    TERM="$TERM"                \
    PS1='(lfs chroot) \u:\w\$ ' \
    PATH=/bin:/usr/bin:/sbin:/usr/sbin:/tools/bin \
    /tools/bin/bash --login +h
~~~

> 无法更改root密码的问题：
> 虚拟系统中需要先把/dev挂起来 mount -v --bind /dev $LFS/dev

### 编译内核



### 引导起来

linux的启动过程

~~~

~~~

- initrd需要指定, book中不指定是觉得这是需要自己领悟?

自己编译的内核启动用vmware引导后识别不到"/dev/sda1", 后来把宿主系统的内核和initrd都拷贝过来用，能够启动成功，
是因为自己编译的内核没有scsi支持模块

修改lfs的磁盘为首个磁盘, 启动就好直接拉起

或者使用qemu模拟器，它原生支持vmdk，支持可以启动

~~~
qemu-system-x86_64 lfs.vmdk
~~~

### 用UEFI引导



### 资料

中文文档:
https://lctt.github.io/LFS-BOOK/lfs-systemd/index.html