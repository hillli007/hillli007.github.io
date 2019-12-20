---
title: android-wms
date: 2018-10-15 00:19:31
tags:
categories: Android框架
---

###	工作方式

WMS内部工作方式采用的是惯例：Android消息驱动模型

![wms-working.png](https://hillli-1253216866.cos.ap-shenzhen-fsi.myqcloud.com/blog/android-framework/wms-working.png)


### 1.2	WMS的IPC架构  

#### 本地代理  

应用开发者通过WindowManager类和WMS打交道，比如上面实例的API：**addView**

getSystemService实际返回的是WindowManager的实现WindowManagerImpl:

~~~java
registerService(WINDOW_SERVICE, new ServiceFetcher() {
        public Object getService(ContextImpl ctx) {
          //…
          return new WindowManagerImpl(display);
        });
｝
~~~

![wms-native.png](https://hillli-1253216866.cos.ap-shenzhen-fsi.myqcloud.com/blog/android-framework/wms-native.png)


WindowManagerImpl的addview实际上调用WindowManagerGlobal唯一实例的addview:

~~~java
@Override
public void addView(View view, ViewGroup.LayoutParams params) {
        mGlobal.addView(view, params, mDisplay, mParentWindow);
}
~~~

WindowManagerGlobal在每个应用中是唯一实例，负责管理该应用的所有窗口：

~~~java
ArrayList<View> mViews                         //窗口View树的根
ArrayList<ViewRootImpl> mRoots                 //管理View树的ViewRoot
ArrayList<WindowManager.LayoutParams> mParams  //窗口属性
~~~

WindowManagerGlobal.addView除了记录View，还将为View新建一个ViewRoot

~~~java
root = new ViewRootImpl(view.getContext(), display);
view.setLayoutParams(wparams);

//添加到全局变量
mViews.add(view);
mRoots.add(root);
mParams.add(wparams);

try {
  //把View记录到ViewRoot的mView中，因为接下来ViewRoot要频繁访问
  root.setView(view, wparams, panelParentView);
} catch (RuntimeException e) {
  //…
}
~~~

#### ViewRoot是什么？

+ ViewRoot是应用本地View系统引擎
+ ViewRoot负责与WMS进行通信
+ ViewRoot通过IWindowSession访问WMS

![wms-communicate.jpg](https://hillli-1253216866.cos.ap-shenzhen-fsi.myqcloud.com/blog/android-framework/wms-communicate.jpg)



在ViewRoot的构造器中：

~~~java
public ViewRootImpl(Context context, Display display) {
    Context = context;
    //获取连接
    mWindowSession = WindowManagerGlobal.getWindowSession();
    //…
    //IWindow对象，WMS将通过它管理
    mWindow = new W(this);
    //…
}
~~~

#### IWindowSession是什么？

ViewRoot其实还是通过WM服务来获取IWindowSession，再通过IWindowSession来访问WMS，而不是直接访问WMS

WindowManagerGlobal.getWindowSession()的实现：

~~~java
public static IWindowSession getWindowSession() {
    IWindowManager windowManager = getWindowManagerService();
    sWindowSession = windowManager.openSession();  //划重点                       
    return sWindowSession;
}
~~~

IWindowSession在服务端的实现是Session，Seesion.java实现如下：

~~~java
class Session extends IWindowSession.Stub{
    @Override
    public int addToDisplay(IWindow window, …) {
          return mService.addWindow(this, window, …);
    }
}
~~~


#### IWindow是什么？

ViewRoot通过IWindowSeession.addToDisplay向WMS添加mWindow（W对象，IWindow子类），使WMS也可以访问Window，实现双向通信。

~~~java
public void setView(View view, WindowManager.LayoutParams attrs, View panelParentView)
{
    //…
    res = mWindowSession.addToDisplay(mWindow, …);
    //…
}
~~~

#### 总结

到此，整个调用流程就出来了：

![wms-relation.png](https://hillli-1253216866.cos.ap-shenzhen-fsi.myqcloud.com/blog/android-framework/wms-relation.png)


1. WindowManager.addview真正实现是WindowManagerGlobal.addView；
2. WindowManagerGlobal.addView创建ViewRoot，保存view、viewroot、params；
3. 然后调用ViewRoot.setView，既调用WindowSession.addToDisplay；
4. WindowSession.addToDisplay在服务端Session的实现：WMS.addWindow；


#### 添加窗口

主要成员变量

* mSessions: WindowSession列表，与各窗口通信
* mTokenMap: WindowToken列表，一个token关联一系列有联系的窗口
* mWindowMap: WindowState列表，代表窗口
* mPolicy: 即 PhoneWindowManager

addWindow的具体流程：

1. 权限检查
2. WMS保存一个 窗口-状态 的键值对表，检查是否重复添加
3. 如果是子窗口，那么它的父窗口不能也是子窗口
4. 根据不同窗口类型，确定是否加WindowToken（窗口令牌）
5. 为新增的窗口新建一个WindowState
6. 如果客户端死亡，则不再执行
7. 窗口参数调整
8. 如果新增了WindowToken,添加到全局表
9. 重新调整窗口顺序
10. 计算窗口大小
11. 分配最终层级值
