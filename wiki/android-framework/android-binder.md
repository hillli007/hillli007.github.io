---
title: Binder
date: 2018-10-15
tags:
categories: Android
---

### AIDL

最常见的Binder应用就是AIDL了，AIDL让IPC通讯就像进程内的调用函数那样简单。自动的帮你完成了参数序列化发送以及解析返回数据的那一系列麻烦。而需要做的就是写上一个接口文件，然后利用aidl工具转化一下得到另一个java文件，这个文件在服务和客户端程序各放一份。服务程序继承IxxxxService.Stub 然后将函数接口里面的逻辑代码实现一下。开发者甚至完全感觉不到Binder的存在。

### 直接使用Binder

直接使用Binder需要写大量的参数序列化代码，这个工作是很枯燥的，因此框架内的大多数服务也是用的AIDL。但是ActivityManager是其中一个例外，这里我们通过它来分析一下Java上的Binder使用

#### 服务端

首先看继承关系 ActivityManagerService -> ActivityManagerNative -> Binder，这么看来Binder就是服务端的实现(参考AIDL的Stub)

#### 客户端

那么再看看本地代理是什么：

例如调用ActivityManager的forceStopPackage就是调用的ActivityManagerNative的forceStopPackage

~~~
public void forceStopPackageAsUser(String packageName, int userId) {
    try {
        ActivityManagerNative.getDefault().forceStopPackage(packageName, userId);
    } catch (RemoteException e) {
        throw e.rethrowFromSystemServer();
    }
}
~~~

ActivityManagerNative.getDefault()获取的是一个单例的代理接口

~~~
private static final Singleton<IActivityManager> gDefault = new Singleton<IActivityManager>() {
    protected IActivityManager create() {
        IBinder b = ServiceManager.getService("activity");
        IActivityManager am = asInterface(b);
        return am;
    }
};
~~~

从ServiceManager那里查询"activity"对应的IBinder，通过asInterface获取真正能用的代理！而asInterface就是调用IBinder的queryLocalInterface

~~~
static public IActivityManager asInterface(IBinder obj) {
    if (obj == null) {
        return null;
    }

    IActivityManager in = (IActivityManager)obj.queryLocalInterface(descriptor); // !!!
    if (in != null) {
        return in;
    }

    return new ActivityManagerProxy(obj);
}
~~~


IActivityManager接口就是客户端调用和服务端实现的约束，在SystemServer中它就是AMS，在APP进程里它就是ActivityManagerProxy

它继承IInterface，这就要求服务端实现和客户端代理都必须实现方法: IBinder asBinder();

#### 总结

在java层实质就是:

1. 客户端调 IBinder.transcat 传入reply的引用作为参数
2. 服务端在 Binder.onTranscact 响应，对传入的reply参数写值

### Messenger

AIDL隐藏了Binder细节，我们约定接口就行，但是有时候我们甚至不想约定接口。对的，谷歌也料想到了，于是就允许我直接向其他进程发送可序列化的 Message。
但其实Messenger只是包装了IMessenger，它本质和AIDL是没有任何区别的，而IMessenger的接口只有一个方法：

~~~
void send(Message message)
~~~

使用方法：

- 服务端：用 Handler 构造 Messenger(Handler h)， 在 Handler 里处理发过来的消息
- 客户端：用获得的 IBinder 构造 Messenger(IBinder b)，调用send来发消息

这样看来Messenger只能从Client到Service的单向通信，但是Client的Messenger可以通过Message的replyTo传递给Service，这样就可以实现双向通信了。
这里传的是Messenger而不是直接传IBinder,其实Message的writeToParcel方法中还是序列化的Binder。

~~~
public static void writeMessengerOrNullToParcel(Messenger messenger,Parcel out) {
    out.writeStrongBinder(messenger != null ? messenger.mTarget.asBinder() : null);
}
~~~

当然读的时候是用 IBinder 来new一个Messenger

~~~
public static Messenger readMessengerOrNullFromParcel(Parcel in) {
    IBinder b = in.readStrongBinder();
    return b != null ? new Messenger(b) : null;
}
~~~

### AsyncChannel (待续)

