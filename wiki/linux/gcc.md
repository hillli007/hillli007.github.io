---
title: gcc
date: 2018-10-14 23:45:50
tags:
---

## 编译四个步骤

先从一个最简单的例子出发

~~~
#include <stdio.h>

int main(int argc, char **argv)
{
    printf("Here is main! \n");
    return 0;
}
~~~

1.预处理，生成预编译文件（.i文件）

~~~
gcc -E foo.c -o foo.i
~~~

2.编译，生成汇编代码（.s文件）

~~~
gcc -S foo.i -o foo.s
~~~

3.汇编，生成目标文件（.o文件）,-c表示只编译(compile)源文件但不链接

~~~
gcc -c foo.s -o foo.o
~~~

4.链接，生成可执行文件

~~~
gcc foo.o -o foo
~~~

不指定参数可以直接编译出可执行文件: gcc hello.c -o hello

~~~
gcc foo.c -o foo
~~~

## Include

## 链接

用ldd可以检查二进制程序的链接库使用情况，如果找不到可以指定LD_LIBRARY_PATH

