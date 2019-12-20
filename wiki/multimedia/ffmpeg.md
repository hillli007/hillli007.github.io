---
title: FFMPEG
date: 2018-10-16
tags:
categories: 多媒体技术
---

ffmpeg基本框架和使用，以及交叉编译树莓派和安卓上可用的动态库的方法

## Linux（x86）平台

### 编译安装

~~~
sudo apt-get install yasm
git clone https://git.ffmpeg.org/ffmpeg.git
./configure --prefix=host --enable-shared --disable-static --disable-doc
make
make install
~~~

yasm用于加速编译，configure选项用help查询。

### 安装ffplay

#### 编译安装SDL2

~~~
#依赖库
sudo apt-get install libasound2-dev

#下载源码
wget http://www.libsdl.org/release/SDL2-2.0.5.zip

#编译安装
unzip SDL2-2.0.5.zip
cd SDL2-2.0.5/
./configure --prefix=/usr/local/
make
sudo make install
~~~

开启--enable-ffplay选项

~~~
./configure --prefix=host --enable-shared --disable-static --disable-doc --enable-ffplay
make
make install
~~~

### 测试

~~~
g++ -I ../include/ test.cpp -o test.exe -L../lib/ -lavcodec -lavdevice -lavfilter -lavformat -lavutil -lswresample -lswscale
export LD_LIBRARY_PATH=../lib/
./test.exe
~~~

### 最简单的视频播放器(SDL2)

~~~

~~~

## Android

### 交叉编译lib

大坑:最新的NDK版本，需要standalone ndk

~~~
~~~

解决编译错误

### 与java层对接
