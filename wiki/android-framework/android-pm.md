---
title: Android包管理与权限机制
date: 2018-10-15 00:05:17
tags:
categories: Android框架
---

### 打包

- aapt打包资源文件(res/ assets/ AndroidManifest.xml)，生成resources.ap_ 和R.java文件
- 处理AIDL文件，生成对应的.java文件
- javac 编译Java文件，生成对应的.class文件
- 把.class文件转化成Davik VM支持的.dex文件
- apkbuilder：打包生成未签名的.apk文件
- jarsigner：对未签名.apk文件进行签名
- zipalign：对签名后的.apk文件进行对齐处理

### 混淆

- 混淆器(Proguard)通过删除从未用过的代码和使用晦涩名字重命名类、字段和方法，对代码进行压缩，优化和混淆。Proguard被集成在android 构建系统中。

- 不是所有Java代码都可以混淆，比如需要加入了so文件，需要调用里面的方法，那么调用JNI访问so文件的方法就不能被混码。在导出的时候，可能不会报错。但是在手机上运行的时候，需要调用so文件的时候，就会报某某方法无法找到。这个时候就需要用到配置文件proguard-project.txt，配置文件可另外指定。

配置文件例子：

所有View的子类及其子类的get、set方法都不进行混淆

~~~
-keepclassmembers public class * extends android.view.View {        void set*(***);       *** get*();}
~~~

### UID和GID

- 每个程序在安装时都会指定一个Linux用户ID（UID），并且在设备中保持它的持久性
- 安卓中应用程序用户ID(UID)等于用户组ID(GID)
- 可以在AndroidManifest文件的manifest标签属性sharedUserId指定用户ID
- 应用可分配相同的用户ID，但必须有相同签名
- 拥有同一个UID的多个APK可以配置成运行在同一个进程中，所以默认就是可以互相访问任意数据
- 拥有同一个UID的多个APK也可以配置成运行成不同的进程, 可以访问其他APK的数据目录下的数据库和文件，就像访问本程序的数据一样

### GIDS

- GIDS是由框架在APK安装过程中生成，与APK申请的具体权限相关
- 如果 APK 申请的相应的 permission 被 granted ，而且权限有对应的gids，那么这个APK的GIDS中将包含这个权限的GIDS
- 一个GIDS相当于一个权限的集合一个UID可以关联GIDS，表明该UID拥有多种权限

### 系统应用和特权应用

特权app首先是System app，然后要具有ApplicationInfo.PRIVATE_FLAG_PRIVILEGED标志

特定的shared uid的app是特权应用，同时也是系统应用

- android.uid.system
- android.uid.phone
- android.uid.log
- android.uid.nfc
- android.uid.bluetooth
- android.uid.shell

特定目录中的app:

- /vendor/overlay
- /system/framework
- /system/priv-app
- /system/app
- /vendor/app
- /oem/app

特定目录中的app

- /system/priv-app
- /system/framework(该目录中的apk，只是包含资源，不包含代码）


### 签名

1. APK必须签名才能安装，必须签名相同才能覆盖升级
2. 具有相同签名的程序可运行在同一个进程
3. Android权限机制在某个权限(permission)的protectionLevel是signature时，允许同签名程序共享数据和功能

## Permission创建

### 读取

从AndroidManifest.xml中提取permission信息

### 验证

获取Package中的证书，验证，并将签名信息保存在Package结构中

如果该package来自 system img （系统app），那么只需要从该 Package 的AndroidManifest.xml 中获取签名信息，而无需验证其完整性。但是如果这个 package 与其它 package 共享一个 uid ，那么这个共享 uid 对应的 sharedUser 中保存的签名与之不一致，那么签名验证失败。

如果是普通的 package ，那么需要提取证书和签名信息，并对文件的完成性进行验证

### 清除领养

如果是普通的package，那么清除package的mAdoptPermissions字段信息，系统 package 升级才使用

### 分配uid

如果在 AndroidManifest.xml 中指定了 shared user ，那么先查看全局 list 中（ mSharedUsers ）是否该 uid 对应的 SharedUserSetting 数据结构，若没有则新分配一个 uid ，创建 SharedUserSetting 并保存到全局全局 list （ mSharedUsers ）中。

mUserIds 保存了系统中已经分配的 uid 对应的 SharedUserSetting 结构。每次分配时总是从第一个开始轮询，找到第一个空闲的位置 i ，然后加上 FIRST_APPLICATION_UID 即可。

### 维护数据结构

创建 PackageSettings 数据结构。并将 PackageSettings 与 SharedUserSetting 进行绑定

其中 PackageSettings 保存了 SharedUserSetting 结构；而 SharedUserSetting 中会使用 PackageSettings 中的签名信息填充自己内部的签名信息，并将 PackageSettings 添加到一个队列中，表示 PackageSettings 为其中的共享者之一。
在创建时，首先会以 packageName 去全局数据结构 mPackages 中查询是否已经有对应的 PackageSettings 数据结构存在。如果已经存在 PackageSettings 数据结构（比如这个 package 已经被 uninstall ，但是还没有删除数据，此时 package 结构已经被释放）。那么比较该 package 中的签名信息（从 AndroidManifest 中扫描得到）与 PackageSettings 中的签名信息是否匹配。如果不匹配但是为 system package ，那么信任此 package ，并将 package 中的签名信息更新到已有的 PackageSettings 中去，同时如果这个 package 与其它 package 共享了 uid ，而且 shared uid 中保存的签名信息与当前 package 不符，那么签名也验证失败。

### 权限领养

如果 mAdoptPermissions 字段不为空，那么处理 permission 的领养，从指定的 package 对应的 PackageSettings 中，将权限的拥有者修改为当前 package ，一般在 system app 升级的时候才发生，在此之前需要验证当被领养的 package 已经被卸载，即检查 package 数据结构是否存在

### 添加自定义权限

将 package 中定义的 permissionGroup 添加到全局的列表 mPermissionGroups 中去；
将 package 中定义的 permissions 添加到全局的列表中去
如果是 permission-tree添加到 mSettings.mPermissionTrees
如果是permission 添加到 mSettings.mPermissions

### 清除不一致的 permission 信息

清除不一致的 permission-tree 信息。如果该 permission-tree 的 packageSettings 字段为空，说明还未对该 package 进行过解析（若代码执行到此处时 packageSettings 肯定已经被创建过），将其 remove 掉。如果 packageSettings 不为空，但是对应的 package 数据结构为空（说明该 package 已经被卸载，但数据还有保留），或者 package 数据结构中根本不含有这个 permission-tree ，那么将这个 permission-tree 清除
清除不一致的 permission 信息。如果 packageSettings 或者 package 结构为空（未解析该 package 或者被卸载，但数据有保留），或者 package 中根本没有定义该 permission ，那么将该 permission 清除

### 授权

对每一个 package 进行轮询，并进行 permission授权

对申请的权限进行检查，并更新 grantedPermissions 列表
如果其没有设置 shared user id ，那么将其 gids 初始化为 mGlobalGids ，它从 permission.xml 中读取

遍历所有申请的权限，进行如下检查：

如果是该权限是 normal 或者 dangerous的。通过检查。
如果权限需要签名验证。如果签名验证通过。还需要进行如下检查
如果程序升级，而且是 system package 。那么是否授予该权限要看原来的 package 是否被授予了该权限。如果被授予了，那么通过检查，否则不通过。
如果是新安装的。那么检查通过。

如果检查通过，那么将这个 permission 添加到 package 的 grantedPermissions 列表中，表示这个 permission 申请成功（ granted ）。申请成功的同时会将这个申请到的 permission 的 gids 添加到这个 package 的 gids 中去
