---
title: git & gerrit
date: 2018-10-15 00:02:11
tags:
categories: Linux
---

## 分支和标签

## 子模块

### add

添加子模块，会自动clone:

~~~
git submodule add <url> [path]
~~~

可以递归克隆仓库，子模块也会被克隆下来

~~~
git clone project.git projectDir --recursive
~~~

### init

### 内部原理

在 .git/config下会添加submodule项

~~~
[submodule "third-party/GCanvas"]
        url = https://github.com/alibaba/GCanvas.git
        active = true
~~~

在 .git/modules 下存放子模块的 .git 文件夹，子模块目录下只有.git文件，内容是.git文件夹路径。

~~~
gitdir: ../../.git/modules/third-party/GCanvas
~~~

远程仓库都是靠 .gitsubmodule 文件来记录子模块的，具体本地的.git文件夹的东西不影响

.git文件夹应该是 git submodule命令检测依据，
手动编辑 .gitsubmodule文件，init和update都无效，只能从新Add

