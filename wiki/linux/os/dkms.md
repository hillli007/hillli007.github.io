---
title: DKMS - 内核模块管理
date: 2018-10-14 23:46:44
tags:
categories: Linux 操作系统
---

## 引文

由于工作环境是ubuntu，想在电脑上装上无线网卡，于是去找网卡的驱动源码:

[rtl8192eu github地址]([https://github.com/Mange/rtl8192eu-linux-driver)

发现里面用了dkms工具，在这里记录一下。

## DKMS

### 什么是dkms

DKMS全称是DynamicKernel ModuleSupport，它可以帮我们维护内核外的驱动程序，在内核版本变动之后可以自动重新生成新的模块

在使用dkms之前首先需要确保系统中已经安装了DKMS，在Ubuntu下可以执行下面这个命令安装:

~~~
sudo apt-get install dkms
~~~

### dkms作用

我们都知道，如果要使用没有集成到内核之中的Linux驱动程序需要手动编译。当然，这并不是一件什么难事，即使是对于没有编程经验的Linux使用者，只要稍微有点hacker的意识，努力看看代码包里的Readme或者INSTALL文件，按部就班的执行几条命令还是很容易办到的。但这里还有一个问题，Linux模块和内核是有依赖关系的，如果遇到因为发行版更新造成的内核版本的变动，之前编译的模块是无法继续使用的，我们只能手动再编译一遍。这样重复的操作有些繁琐且是反生产力的，而对于没有内核编程经验的使用者来说可能会造成一些困扰，使用者搞不清楚为什么更新系统之后，原来用的好好的驱动程序突然就不能用了。这里，就是Dell创建的DKMS项目的意义所在。DKMS可以帮我们维护内核外的这些驱动程序，在内核版本变动之后可以自动重新生成新的模块。

### dkms使用

DKMS要求我们的代码目录必须以-的格式命名，主要命令如下所示:

- status查看管理状态
- add添加模块
- build编译模块
- install安装模块
- uninstall卸载模块
- remove删除模块

### 编写dkms.conf

要使用DKMS管理模块，需要在源代码下面包含dkms.conf文件，在源代码根目录下面创建并配置dkms.conf

~~~
PACKAGE_NAME="hello"
PACKAGE_VERSION="0.1"
AUTOINSTALL="yes"
CLEAN="make clean"
BUILT_MODULE_NAME[0]="hello"
DEST_MODULE_LOCATION[0]="/updates"
MAKE[0]="make all KVERSION=$kernelver"
~~~

PACKGE_NAME用于指示整个包的模块，必须要设定。必须要有编译的模块和这个参数同名，否则将提示找不到二进制包，所以可以指定为编译的任何一个模块名。
AUTOINSTALL当更改kernel的时候会自动更新模块
BUILT_MODULE_NAME[0]当有多个模块时，可以按序号设定每个模块的参数，注意模块名后面不需要加后缀。
  
### 添加模块

首先将代码命名为module-version格式并复制到/usr/src/下面，执行如下命令添加源代码到dkms。

~~~
sudo dkms add rtl8192eu/1.0
~~~

接下来可以执行如下命令编译和安装。

~~~~
sudo dkms build rtl8192eu/1.0
sudo dkms install rtl8192eu/1.0
~~~

### 删除模块

如果不需要使用模块，可以用如下命令删除。

~~~
sudo dkms remove rtl8192eu/1.0
~~~

### 制作deb包

使用DKMS更为常见的用法是制作deb包，用户可以直接从deb包安装，制作deb包需要安装如下工具。

~~~
sudo apt-get install dh-make libdigest-md5-file-perl
~~~

制作deb包的命令如下

~~~
sudo dkms mkdeb rtl8192eu/1.0
~~~
