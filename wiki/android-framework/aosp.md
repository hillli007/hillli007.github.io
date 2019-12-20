---
title: AOSP
date: 2018-10-15
tags:
categories: Android系统
---

## 源码准备

### 准备repo工具

- [Repo使用](http://source.android.com/source/using-repo.html)
- [清华大学开源镜像站](https://mirror.tuna.tsinghua.edu.cn/help/git-repo)

~~~
mkdir ~/bin
export PATH=~/bin:$PATH
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
chmod a+x ~/bin/repo
~~~

指定清华源加快速度

~~~
export REPO_URL='https://mirrors.tuna.tsinghua.edu.cn/git/git-repo/'
~~~


### 拉取

~~~
#官方仓库: https://android.googlesource.com/platform/manifest
#清华源: https://aosp.tuna.tsinghua.edu.cn/platform/manifest
repo init -u manifest-url -b branch/tag
repo sync
repo start branch-name --all
~~~

### manifest

repo库由清单文件管理，清单文件本身使用一个git库管理，repo init这一步其实就是确定好这个库的地址和分支，如果不用"-m"指定清单
文件，那么库要包含一个default.xml

~~~
<?xml version="1.0" encoding="utf-8"?>
<manifest>
<remote fetch="ssh://lixiaolong@lixiaolong-pc/home/lixiaolong/xxx" name="origin" review=""/>
<default remote="origin" revision="master" sync-j="1"/>

<!-- 如果删掉，对应路径也会删掉 -->
<project name="p1.git" path="prj1"/>
<project name="p2.git" path="prj2"/>
</manifest>
~~~

### 编译完整镜像

要编译完整镜像，需要checkout机型对应的tag，并且下载驱动

- [AOSP官网](https://source.android.com)
- [查询对应的版本](https://source.android.com/source/build-numbers.html#source-code-tags-and-builds)
- [新的版本对应](https://source.android.google.cn/setup/start/build-numbers)
- [驱动不开源，需下载](https://developers.google.com/android/drivers)
- [下载官方镜像(带GMS)](https://developers.google.com/android/images)

## 编译

### 编译环境依赖

Ubuntu 14 以上

~~~
sudo apt-get install git-core gnupg flex bison gperf build-essential zip curl zlib1g-dev\
 gcc-multilib g++-multilib libc6-dev-i386 lib32ncurses5-dev x11proto-core-dev libx11-dev\
 lib32z-dev libgl1-mesa-dev libxml2-utils xsltproc unzip
~~~

Arch Linux

~~~
pacman -S git gnupg flex bison gperf sdl wxgtk squashfs-tools curl ncurses zlib schedtool perl-switch zip unzip

# 确保配置了multilib仓库
pacman -S lib32-zlib lib32-ncurses lib32-readline gcc-libs-multilib gcc-multilib lib32-gcc-libs
~~~

### 环境配置

~~~
# Java
export JAVA_HOME=/path/to/java
export JAVA_JRE_HOME=$JAVA_HOME/jre
export CLASSPATH=$JAVA_HOME/jre/lib
export PATH=$JAVA_HOME/bin:$JAVA_JRE_HOME/bin:$PATH

# Android sdk
export ANDROID_HOME=/path/to/android
export ANDROID_SDK_PLATFORM_TOOLS=$ANDROID_HOME/platform-tools
export ANDROID_SDK_TOOLS=$ANDROID_HOME/tools
export ANDROID_BUILD_TOOLS=/path/to/buildtools
export PATH=$ANDROID_SDK_PLATFORM_TOOLS:$ANDROID_SDK_TOOLS:$ANDROID_BUILD_TOOLS:$PATH

# Android NDK
export NDK=/path/to/ndk
export PATH=$NDKROOT:$PATH

# Custom path
export PATH=~/bin:$PATH
~~~

### 集成第三方产品

#### 集成GMS: 

[开源的openapps](https://github.com/opengapps/aosp_build) ：包括基础服务、配置、APK等

在devices.mk下调用:

~~~
$(call inherit-product-if-exists, vendor/gms-packs/products/gms.mk)
~~~


### 编译问题记录

* https://stackoverflow.com/questions/49955137/error-when-build-lineageos-make-ninja-wrapper-error-1

### IDE配置

调节启动参数：studio64.exe.vmoptions

~~~
-Xms128m JVM初始分配的堆内存
-Xmx512m JVM最大允许分配的堆内存，按需分配
~~~

用源码自带工具生成配置文件

~~~
# 生成新的 idegen.jar 文件
source build/envsetup.sh
mmm development/tools/idegen

# 生成idea配置文件
development/tools/idegen/idegen.sh
~~~

各个配置文件作用：

| 文件        | 作用                                                                                          |
| ----------- | --------------------------------------------------------------------------------------------- |
| android.iws | 包含工作区的个人设置，比如打开过的文件，版本控制工具的配置，本地修改历史，运行和debug的配置等 |
| android.ipr | 一般保存了工程相关的设置，比如modules和modules libraries的路径，编译器配置，入口点等          |
| android.iml | 用来描述modules。它包括modules路径、 依赖关系，顺序设置等。一个项目可以包含多个 \*.iml 文件   |