---
title: Makefile
date: 2018-10-14 23:46:11
tags:
---

### GNU 基础

#### 变量

定义：

+ "="  替换,右值在整个Makefile展开后决定
+ ":=" 替换,右值取当前值
+ "+=" 追加
+ "?=" 空则赋值

~~~
x=foo
y=$(x) bar
x=xyz
~~~

结果: y == xyz bar

~~~
x=foo
y:=$(x) bar
x=xyz
~~~

结果: y == foo bar


#### 函数

调用: $(call funcname, args...)

* 打印Log $()

~~~
define a_func
# content
endef
~~~