### 32/64 位

1. 32/64位进程分别是zygote和zygote64的子进程；
2. 64位CPU的手机可以运行32位进程,如apk没有arm64-v8a；
3. gradle中abiFilter配置的是是否打包对应abi的so库，尽管有些模块已经编译出来最终也可能不打包进去；
4. 默认打包路径是jniLibs，可设置： jniLibs.srcDirs=['d1','d2']

### 编译工具

交叉编译是准备好目标机器的

- 工具链:
- sysroot:

### ndk-build

- Application.mk:
- jni/Android.mk:
- jni/src:
- libs:
- AndroidManifest:

### CMake

### 独立工具链

很多第三方的C/C++库不提供Android的编译方案，

~~~
$NDK/build/tools/make-standalone-toolchain.sh \
        --arch=arm \
        --platform=android-21\
        --install-dir=/path/to/my-android-toolchain
~~~

以freetype为例子

~~~
export PATH=/path/to/my-android-toolchain/bin/:$PATH
export CC=aarch64-linux-android-gcc
export CXX=aarch64-linux-android-g++
./configure \
    --disable-static \
    --enable-shared \
    --host=arm-linux-androideabi --prefix=/freetype \
    --without-zlib
make
make install DESTDIR=$(pwd)
~~~

## 各种坑

[ndk r17b版本后不再支持mips](https://www.jianshu.com/p/76ed525ec1be)

~~~
android {
    ...
    packagingOptions{  
        doNotStrip '*/mips/*.so'  
        doNotStrip '*/mips64/*.so'  
    } 
}
~~~

[JNI在Windows环境下因长路径导致编译失败](https://www.jianshu.com/p/86ed53fc242a)

### 参考资料

- [官网](https://developer.android.com/ndk/guides)
- [官方文档](https://developer.android.google.cn/studio/projects/add-native-code.html)
- [官方Demo](https://github.com/googlesamples/android-ndk.git)

### 一些博客

https://fucknmb.com/2017/06/27/cmake-%E4%BA%A4%E5%8F%89%E7%BC%96%E8%AF%91/
https://android.googlesource.com/platform/tools/cmake-utils/+/cmake-master-dev/android.toolchain.cmake