## ANR

https://developer.android.com/topic/performance/vitals/anr#java

~~~mermaid
graph LR;
    a[ANR]-->a1[KeyDispatchTimeout];
    a-->a2[BroadcastReceiver];
    a-->a3[Service];
    a-->a4[ContentResolver];
~~~

### 4种ANR类型

主线程对输入事件在5秒内没有处理完毕

主线程在执行BroadcastReceiver的onReceive函数时10秒内没有执行完毕

主线程在执行Service的各个生命周期函数时20秒内没有执行完毕，是低概率错误

ContentResolver进程可以触发AMS汇报ContentProvider所在进程的ANR

~~~
public static ContentProviderClient acquireUnstableProviderOrThrow(ContentResolver resolver, String authority) throws RemoteException {
    final ContentProviderClient client = resolver.acquireUnstableContentProviderClient(authority);
    if (client == null) {
        throw new RemoteException("Failed to acquire provider for " + authority);
    }
    client.setDetectNotResponding(PROVIDER_ANR_TIMEOUT);
    return client;
}
~~~

### Trace文件

由AMS.dumpStackTraces生成, Trace文件就是记录的是ANR发生时的线程状态. Java线程状态和虚拟机中的线程状态不是一致的，但它们是有对应关系的:

| Thread.java中定义的状态 | Thread.cpp中定义的状态 | 说明                                      |
| ----------------------- | ---------------------- | ---------------------------------------|
| TERMINATED              | ZOMBIE                 | 线程死亡，终止运行                        |
| RUNNABLE                | RUNNING/RUNNABLE       | 线程可运行或正在运行                      |
| TIMED_WAITING           | TIMED_WAIT             | 执行了带有超时参数的wait、sleep或join函数 |
| BLOCKED                 | MONITOR                | 线程阻塞，等待获取对象锁                  |
| WAITING                 | WAIT                   | 执行了无超时参数的wait函数                |
| NEW                     | INITIALIZING           | 新建，正在初始化，为其分配资源            |
| NEW                     | STARTING               | 新建，正在启动                            |
| RUNNABLE                | NATIVE                 | 正在执行JNI本地函数                       |
| WAITING                 | VMWAIT                 | 正在等待VM资源                            |
| RUNNABLE                | SUSPENDED              | 线程暂停，通常是由于GC或debug被暂停       |
|                         | UNKNOWN                | 未知状态                                  |