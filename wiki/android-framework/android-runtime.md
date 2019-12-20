---
title: android-runtime
date: 2018-10-15 00:05:45
tags:
categories: Android框架
---

## app_process

- 根据预设配置和启动参数启动JVM
- 初始化AndroidRuntim: 初始化Binder, 加载framework等
- 从Native层启动一个Java类，ZygoteInit或者RuntimeInit，并且调用main方法;

~~~
AppRuntime runtime(argv[0], computeArgBlockSize(argc, argv));
// ....
if (zygote) {
    runtime.start("com.android.internal.os.ZygoteInit", args, zygote);
} else if (className) {
    runtime.start("com.android.internal.os.RuntimeInit", args, zygote);
}
~~~

### Zygote和SystemServer

Zygote进程就是启动参数带"--zygote"的app_process,也就是运行ZygoteInit，它由init进程拉起

~~~
service zygote /system/bin/app_process32 -Xzygote /system/bin --zygote --start-system-server --socket-name=zygote
~~~

SystemServer在较早阶段就被Zygote进程fork出来，后面详细介绍

### 应用

~~~
base=/system
export CLASSPATH=$base/framework/pm.jar
exec app_process $base/bin com.android.commands.pm.Pm "$@"
~~~

一般应用由Zygote进程拉起，的最后会启动ActivityThread的main方法，后面详细介绍

## Native层：AppRuntime

### start方法

设置虚拟机参数，创建虚拟机

~~~
startVm(&mJavaVM, &env, zygote)
JNI_CreateJavaVM(pJavaVM, pEnv, &initArgs)
~~~

注册JNI方法
~~~
startReg(env)
register_com_android_internal_os_Zygote(env)
jniRegisterNativeMethods()
~~~

调用com.android.internal.os.ZygoteInit类的main函数

~~~
env->CallStaticVoidMethod(startClass, startMeth, strArray);
~~~

## Java层：ZygoteInit

### main方法

~~~
// 创建socket接口，用来和AMS通讯
// 通过/dev/socket/zygote文件描述符来创建socket接口
registerZygoteSocket();

// 启动SystemServer
startSystemServer();

// 进入一个无限循环
// 在前面创建的socket接口上等待AMS请求创建新的应用程序进程
runSelectLoopMode();
~~~


~~~
try {
    // ...
    /* Request to fork the system server process */
    // 这里fork SystemServer进程，Zyg开始分裂
    pid = Zygote.forkSystemServer(/* ... */);
} catch (IllegalArgumentException ex) {
    throw new RuntimeException(ex);
}

/* For child process */
if (pid == 0) {
    // 在前面创建的Socket文件描述符，而这里的子进程用不到，关闭
    zygoteServer.closeServerSocket();
    return handleSystemServerProcess(parsedArgs);
}
~~~

返回值pid等0的地方就是子进程(SystemServer)要执行的路径
即新创建的进程会执行handleSystemServerProcess

### handleSystemServerProcess方法

加载SystemServer的jar包

~~~
final String systemServerClasspath = Os.getenv("SYSTEMSERVERCLASSPATH");
cl = createSystemServerClassLoader(systemServerClasspath,parsedArgs.targetSdkVersion);
Thread.currentThread().setContextClassLoader(cl);
~~~

初始化Binder和调用SystemServer的main方法

~~~
RuntimeInit.zygoteInit(parsedArgs.targetSdkVersion, parsedArgs.remainingArgs, cl);
~~~

### RuntimeInit.zygoteInit

// Binder初始化

~~~
nativeZygoteInit()；

AndroidRuntime::com_android_internal_os_RuntimeInit_nativeFinishInit()

AppRuntime::onZygoteInit ()
~~~

// 调用SystemServer.main

~~~
applicationInit(targetSdkVersion, argv, classLoader);

invokeStaticMain(args.startClass, args.startArgs, classLoader);
~~~

## Application

AMS通过Process.start函数来创建一个新的进程
Process通过openZygoteSocketIfNeeded函数来连接到Zygote进程中的Socket

Process.start函数通过zygoteSendArgsAndGetResult向Zygote发送命令创建新进程


###  handleChildProc()

ActivityThread.main()

~~~
// 绑定主线程的Looper
Looper.prepareMainLooper();

// 进入主线程消息循环
Looper.loop();
~~~

### 类加载器

~~~
BootStrapClassLoader
     |
ExtClassLoader
~~~

如果JVM不指定” -Xbootclasspath”启动参数， BootStrapClassLoader将会加载BOOTCLASSPATH里的jar包

SystemServer将使用SYSTEMSERVERCLASSPATH里的jar包

App将使用BaseDexClassLoader来加载jar/dex/apk里的类

编译时可指定要加载的类包

~~~
PRODUCT_BOOT_JARS:
PRODUCT_SYSTEM_SERVER_JARS:
~~~