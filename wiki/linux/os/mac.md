---
title: Mac
date: 2019-12-2
tags:
categories: linux
---

## 系统安装

### U盘安装盘制作

1. 下载安装器应用

2. 制作启动盘

~~~
sudo /Applications/Install\ macOS\ Sierra.app/Contents/Resources/createinstallmedia
--volume /Volumes/tmp
--applicationpath /Applications/Install\ macOS\ Sierra.app
--nointeraction
~~~

--volume 指定U盘挂载路径

3. 启动, "噹"一声，按住option键

### 制作cdr、dmg镜像

以Mojave为例，创建一个大小为6G的dmg文件
~~~
hdiutil create -o ~/Desktop/Mojave.cdr -size 6g -layout SPUD -fs HFS+J
~~~

挂载dmg文件到指定目录
~~~
hdiutil attach ~/Desktop/Mojave.cdr.dmg -noverify -mountpoint /Volumes/install_build
~~~

将所下载的系统安装app文件写入到上面挂载的虚拟光驱磁盘中（也就是第一步建立的空镜像中）
~~~
sudo /Applications/Install\ macOS\ Mojave.app/Contents/Resources/createinstallmedia --volume /Volumes/install_build --nointeraction
~~~

取消挂载，准备操作文件，载点名已经从原来的install_build更改为Install macOS Mojave
~~~
hdiutil detach "/Volumes/Install macOS Mojave"
~~~

格式转换，将制作好的dmg文件转换为cdr
~~~
hdiutil convert ~/Desktop/Mojave.cdr.dmg -format UDTO -o ~/Desktop/Mojave.iso·
~~~

