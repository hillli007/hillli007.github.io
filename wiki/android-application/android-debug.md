## 断点调试

断点调试是指在进程运行在某段代码时停下来，然后在停下来的点上接着一行一行地执行代码(Step Over),或者进入方法的具体实现(Step Into)。
从而能看到它运行的代码和流程，以及当前的变量状态。

### JDWP (Java Debug Wire Protocol)

**JDWP** 是 **Dalvik VM** 的一个线程, 可以建立在adb或者tcp基础上, 与DDMS或Debugger进行通信。

一般IDE自带调试隐藏了这个细节, 实际上还可以用jdb这个个交互式shell的调试工具来通信JDWP线程。

首先需要设置adb转发: `adb forward tcp:<port> jdwp:<process pid>`

具体命令：`adb forward tcp:8700 jdwp:$PID`

?> PS: 使用 `adb jdwp` 就能显示所有支持jdwp调试的进行进程 pid。

以下命令进入jdb调试Shell：

使用所列参数值通过指定的连接器连接到目标 VM：

~~~
jdb -connect <connector-name>:<name1>=<value1>,...
~~~

具体命令：

~~~
jdb -connect com.sun.jdi.SocketAttach:hostname=localhost,port=8700
~~~

## Gradle 问题调试