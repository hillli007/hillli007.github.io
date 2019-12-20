---
title: cmake
date: 2018-10-16 14:41:18
tags:
---

## 基本流程

cmake 没大小写的区分
camke_ 前缀的函数和变量都和cmake本身相关

1. cmake_minimum_required(VERSION 2.6)

2. project( name )
- name_BINARY_DIR: 产物文件夹
- name_SOURCE_DIR

3. set( VAR value )
- CMAKE_ 变量可设置cmake特性
- -DXX=xx : 编译时可用-D选项指定，效果类似set

4. add_executable() / add_library()

include_directories
link_directories
target_link_directories
set_target_properties

## AutoConf

linux下Cmake会生成MakeFile