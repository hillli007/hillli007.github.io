---
title: Android性能相关
date: 2019-03-02
tags:
categories: Android
---

### Systrace

其实是chrome的功能，通过 chrome:tracing 打开

### TraceView

列名    |   描述
------ | ------ 
Name           | 该线程运行过程中所调用的函数名
Incl Cpu Time  | 某函数占用的CPU时间，包含内部调用其它函数的CPU时间
Excl Cpu Time  | 某函数占用的CPU时间，但不含内部调用其它函数所占用的CPU时间
Incl Real Time | 某函数运行的真实时间（以毫秒为单位），内含调用其它函数所占用的真实时间
Excl Real Time | 某函数运行的真实时间（以毫秒为单位），不含调用其它函数所占用的真实时间
Call+Recur Calls/Total | 某函数被调用次数以及递归调用占总调用次数的百分比
Cpu Time/Call  | 某函数调用CPU时间与调用次数的比。相当于该函数平均执行时间
Real Time/Call | 同CPU Time/Call类似，只不过统计单位换成了真实时间

### Profiler
