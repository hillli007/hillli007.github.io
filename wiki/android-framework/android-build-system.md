---
title: Android构建系统
date: 2018-10-15 00:06:10
tags:
categories: Android系统
---

### 概述

- 说明文档在源码目录： build/core/build-system.html

### 编译的工作原理

#### 单模块编译

~~~
LOCAL_PATH := $(call my-dir)
include $(CLEAR_VARS)
LOCAL_MODULE := ModuleName
LOCAL_SRC_FILES := $(call all-java-files-under, src)
include $(BUILD_PACKAGE)
~~~

调用mmm编译该模块，实际运行以下命令:

~~~
ONE_SHOT_MAKEFILE="$MAKEFILE" make -C $T $DASH_ARGS $MODULES $ARGS
~~~

- 意思是设置ONE_SHOT_MAKEFILE环境变量后调用make
- 那么该main.mk文件即会被引用，Make的Target为$MODULES即all\_modules
- 每一个通过 BUILD_XXX 引用的mk文件都含有 all\_modules 这个Target

### LOCAL变量

变量 | 取值 | 说明
:----|:----|:-----
LOCAL_DEX_PREOPT | true/false | 是否生成odex 
LOCAL_MODULE_TAGS | user eng tests optional | 决定该模块在什么版本编译，tests版本不强制语言本地化
LOCAL_MANIFEST_FILE | | 指定manifest的文件，默认是在mk文件同级目录，编译gradle目录结构项目时可用


### 编译变量

实际是一个mk文件路径，用于编译apk、Java类库、C/C++库、C/C++应用程序等

变量 | 说明
:----|:----
BUILD_JAVA_LIBRARY  | Java动态库,编译时用到，运行时从系统库加载，一般都产物都在system/framework
BUILD_STATIC_JAVA_LIBRARY | Java静态库,编译时直接打包
BUILD_PREBUILT  |
BUILD_MULTI_PREBUILT | 集成已有库

### 集成

### 集成aar

~~~
# 编package时引入aar：
LOCAL_STATIC_JAVA_AAR_LIBRARIES:= <aar alias>
...
include $(BUILD_PACKAGE)

# 引入第三方aar：
include $(CLEAR_VARS)
LOCAL_PREBUILT_STATIC_JAVA_LIBRARIES := <aar alias>:libs/<lib file>.aar
include $(BUILD_MULTI_PREBUILT)
~~~

## 开发

### IDE

https://www.jianshu.com/p/a19dcb06cd53

~~~
mmm development/tools/idegen
development/tools/idegen/idegen.sh
~~~

