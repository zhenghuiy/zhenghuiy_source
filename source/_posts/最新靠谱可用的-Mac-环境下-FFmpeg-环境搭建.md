title: 最新靠谱可用的 Mac 环境下 FFmpeg 环境搭建
tags:
  - FFmpeg
  - mac ffmpeg
categories:
  - FFmpeg
date: 2018-05-13 20:46:00
---
大家好，我是光源。

最近在尝试搭建 FFmpeg 开发环境时遇到一个蛋疼的事，Google 了 N 篇文章竟然没有一篇是可以跑起来的！

少部分教程是给出了自我矛盾的配置（是的，按照贴出来的代码和配置，他自己都跑不起来），大部分教程是看着挺全但忽略了某几个关键的点导致跑不起来，更蛋疼的是碰到报错后错误相关的文章也很少，当然还有一些是年代久远过时了。

于是在成功跑起来后，我将整个搭建过程整理出来，希望可以帮到后面的人。

本文基于 Mac OS X + Android Studio 3.2 + FFmpeg 3.3 + CMake。

文章会分为两部分，第一部分是总结一下碰到的几个坑，这样只是因为报错而无法继续的朋友可以先看看是否可以解决问题；第二部分是搭建过程的完整描述（我特意用另一台电脑测试过，可以完美跑起来）。

## 一、FFmpeg 搭建的常见问题

### 1. NDK 的问题

在编译之前，教程都会让我们修改命令中 NDK 的地址为自己本地的地址，我们当然自然而然地改成了 Android Studio 自带的 ndk-bundle。

然后编译的时候就发现会出现`errno.h: No such file or directory`字样的error。

**这是因为 Android Studio 自带的 NDK 缺少相关的 .h 文件，从网上额外下载 NDK 然后编译时使用就可以解决问题。**（基于 FFmpeg 3.3）


### 2. 编译命令的问题

报错信息形如

```
ffmpeg_build.sh: line 14: ./configure: No such file or directory

ffmpeg_build.sh: line 20: --extra-cflags=-Os -fpic -marm: command not found
```

之类的，请检查一下 Android 编辑脚本的 “/” 后是否有空格。

由于不同系统存在差异，最好找对应系统下他人验证可行的编译命令。

### 3. 编译相关的版本

编译过程有 NDK 版本、Android 版本、FFmpeg 版本、Android Studio 版本等，不同版本存在差异，最好是完全使用教程描述时的版本。

比如编译命令中包含的 Android 版本

```
export SYSROOT=$NDK/platforms/android-21/arch-arm/
```

这里的 `android-21` 改成 26 就不行，因为搜索了这两个文件夹只有 21 中有需要的 .h 文件。

<!--more-->

### 4. 项目构建配置

需要将相关的 .so 文件和 include 文件夹（里面包含一些头文件）都放入 libs 文件夹中，并在 gradle 文件夹中指定 jni 目录，最后编写 CMake 文件，缺少一步就报错

## 二、FFmpeg 环境配置最佳实践

### 1. 下载 FFmpeg 源码

```
git clone https://git.ffmpeg.org/ffmpeg.git
```

再切到 3.3 分支

```
git checkout -b 3.3 remotes/origin/release/3.3
```

### 2. 修改 configure 文件

在下载好的 ffmpeg 文件夹内可以找到一个 configure 文件，将其中的：

```
SLIBNAME_WITH_MAJOR='$(SLIBNAME).$(LIBMAJOR)'  
LIB_INSTALL_EXTRA_CMD='$$(RANLIB)"$(LIBDIR)/$(LIBNAME)"'  
SLIB_INSTALL_NAME='$(SLIBNAME_WITH_VERSION)'  
SLIB_INSTALL_LINKS='$(SLIBNAME_WITH_MAJOR)$(SLIBNAME)'
```

修改成

```
SLIBNAME_WITH_MAJOR='$(SLIBPREF)$(FULLNAME)-$(LIBMAJOR)$(SLIBSUF)'  
LIB_INSTALL_EXTRA_CMD='$$(RANLIB)"$(LIBDIR)/$(LIBNAME)"'  
SLIB_INSTALL_NAME='$(SLIBNAME_WITH_MAJOR)'  
SLIB_INSTALL_LINKS='$(SLIBNAME)'
```

### 3. 下载 NDK

就如上文所述，Android Studio 自带的 NDK 缺少部分 .h 文件，不确定是否跟 Android Studio 的 版本或者 NDK 版本有关，也不确定是否所有人都这样。但为了简单，还是下载吧（下载下来的的这个 NDK 只为了编译 FFmpeg，不影响之前的 NDK）。

可以选择去官网下载，我找了个国内的 [Android NDK下载(r10d r13b r14b)](https://blog.csdn.net/momo0853/article/details/73898066)

下载适用于 Mac OS X 的 android-ndk-r14b-darwin-x86_64.zip 文件。

### 4. 编写编译脚本

在 ffmpeg 文件夹根目录下创建一个 `ffmpeg_build.sh` 文件（名称跟配置不相关，想改可以改），具体内容如下：

```
#!/bin/bash
make clean
# NDK的路径，根据自己的安装位置进行设置
export NDK=/Users/yanzhenghui/Downloads/android-ndk
export SYSROOT=$NDK/platforms/android-21/arch-arm/
export TOOLCHAIN=$NDK/toolchains/arm-linux-androideabi-4.9/prebuilt/darwin-x86_64
export CPU=arm
export PREFIX=$(pwd)/android/$CPU
export ADDI_CFLAGS="-marm"
function build_one
{
./configure \
    --prefix=$PREFIX \
    --target-os=linux \
    --cross-prefix=$TOOLCHAIN/bin/arm-linux-androideabi- \
    --arch=arm \
    --sysroot=$SYSROOT \
    --extra-cflags="-Os -fpic $ADDI_CFLAGS" \
    --extra-ldflags="$ADDI_LDFLAGS" \
    --cc=$TOOLCHAIN/bin/arm-linux-androideabi-gcc \
    --nm=$TOOLCHAIN/bin/arm-linux-androideabi-nm \
    --enable-shared \
    --enable-runtime-cpudetect \
    --enable-gpl \
    --enable-small \
    --enable-cross-compile \
    --disable-debug \
    --disable-static \
    --disable-doc \
    --disable-asm \
    --disable-ffmpeg \
    --disable-ffplay \
    --disable-ffprobe \
    --disable-ffserver \
    --enable-postproc \
    --enable-avdevice \
    --disable-symver \
    --disable-stripping \
$ADDITIONAL_CONFIGURE_FLAG
sed -i '' 's/HAVE_LRINT 0/HAVE_LRINT 1/g' config.h
sed -i '' 's/HAVE_LRINTF 0/HAVE_LRINTF 1/g' config.h
sed -i '' 's/HAVE_ROUND 0/HAVE_ROUND 1/g' config.h
sed -i '' 's/HAVE_ROUNDF 0/HAVE_ROUNDF 1/g' config.h
sed -i '' 's/HAVE_TRUNC 0/HAVE_TRUNC 1/g' config.h
sed -i '' 's/HAVE_TRUNCF 0/HAVE_TRUNCF 1/g' config.h
sed -i '' 's/HAVE_CBRT 0/HAVE_CBRT 1/g' config.h
sed -i '' 's/HAVE_RINT 0/HAVE_RINT 1/g' config.h
make clean
# 这里是定义用几个CPU编译，我用4个，一般在5分钟之内编译完成
make -j4
make install
}
build_one
```

需要更改的是这个：

```
# NDK的路径，根据自己的安装位置进行设置
export NDK=/Users/yanzhenghui/Downloads/android-ndk
```

改成你本地的路径。

### 5.执行脚本

从命令行进入 ffmpeg 文件夹后，先执行

```
chmod +x ffmpeg_build.sh
```

命令给脚本开启权限，再执行编译脚本：

```
./ffmpeg_build.sh
```

等待编译完成，即可在 ffmpeg 文件夹内看到 android 文件夹，里面包含了我们需要的 .so 文件和对应的头文件

![图一](http://zhenghuiy-blog.qiniudn.com/1111.png)

### 6.配置 ffmpeg 项目

这部分 [Android - FFmpeg & Mac & AndroidStudio & CMake 环境搭建](http://gavinliu.cn/2017/03/05/Android-FFmpeg-Mac-AndroidStudio-CMake-环境搭建/) 这篇文章写得足够好，我就不重复造轮子了，下面摘抄部分权当转载了。（当然，我也全部测试过能跑起来）

#### 6.1 新建项目
新建一个以 CMake 方式构建的 C++ 项目（记得勾选 Include C++ Support）

![图二](http://zhenghuiy-blog.qiniudn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202018-05-13%20%E4%B8%8B%E5%8D%881.26.56.png)

#### 6.2 拷贝文件到 libs 目录下
将编译好的 .so 和头文件拷贝到 libs 目录下：

![图三](http://zhenghuiy-blog.qiniudn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202018-05-13%20%E4%B8%8B%E5%8D%887.07.51.png)

#### 6.3 指定 jniLibs 路径

```
android {
    ...
    sourceSets {
        main {
            jniLibs.srcDirs = ['libs']
        }
    }
}
```

#### 6.4 配置 abiFilters

因为我们只编译出 armeabi-v7a 的 so，所以需要特别指定一下 abi

```
android {
    ...
    defaultConfig {
      ...
      ndk {
          abiFilters 'armeabi-v7a'
      }
    }
}
```

#### 6.5 编写 CMakeLists

```
# For more information about using CMake with Android Studio, read the
# documentation: https://d.android.com/studio/projects/add-native-code.html
# Sets the minimum version of CMake required to build the native library.
cmake_minimum_required(VERSION 3.4.1)
# Creates and names a library, sets it as either STATIC
# or SHARED, and provides the relative paths to its source code.
# You can define multiple libraries, and CMake builds them for you.
# Gradle automatically packages shared libraries with your APK.
add_library( # Sets the name of the library.
             native-lib
             # Sets the library as a shared library.
             SHARED
             # Provides a relative path to your source file(s).
             src/main/cpp/native-lib.cpp )
add_library( avcodec-57
             SHARED
             IMPORTED )
set_target_properties( avcodec-57
                       PROPERTIES IMPORTED_LOCATION
                       ../../../../libs/armeabi-v7a/libavcodec-57.so )
add_library( avfilter-6
             SHARED
             IMPORTED )
set_target_properties( avfilter-6
                       PROPERTIES IMPORTED_LOCATION
                       ../../../../libs/armeabi-v7a/libavfilter-6.so )
add_library( avformat-57
             SHARED
             IMPORTED )
set_target_properties( avformat-57
                       PROPERTIES IMPORTED_LOCATION
                       ../../../../libs/armeabi-v7a/libavformat-57.so )
add_library( avutil-55
             SHARED
             IMPORTED )
set_target_properties( avutil-55
                       PROPERTIES IMPORTED_LOCATION
                       ../../../../libs/armeabi-v7a/libavutil-55.so )
add_library( swresample-2
             SHARED
             IMPORTED )
set_target_properties( swresample-2
                       PROPERTIES IMPORTED_LOCATION
                       ../../../../libs/armeabi-v7a/libswresample-2.so )
add_library( swscale-4
             SHARED
             IMPORTED )
set_target_properties( swscale-4
                       PROPERTIES IMPORTED_LOCATION
                       ../../../../libs/armeabi-v7a/libswscale-4.so )
include_directories( libs/include )
find_library( # Sets the name of the path variable.
              log-lib
              # Specifies the name of the NDK library that
              # you want CMake to locate.
              log )
# Specifies libraries CMake should link to your target library. You
# can link multiple libraries, such as libraries you define in this
# build script, prebuilt third-party libraries, or system libraries.
target_link_libraries( # Specifies the target library.
                       native-lib
                       avcodec-57
                       avfilter-6
                       avformat-57
                       avutil-55
                       swresample-2
                       swscale-4
                       # Links the target library to the log library
                       # included in the NDK.
                       ${log-lib} )
```

#### 6.6 Java 引入 so 库

```
static {
    System.loadLibrary("native-lib");
    System.loadLibrary("avcodec-57");
    System.loadLibrary("avfilter-6");
    System.loadLibrary("avformat-57");
    System.loadLibrary("avutil-55");
    System.loadLibrary("swresample-2");
    System.loadLibrary("swscale-4");
}
```

最后运行测试下，即配置完成。



## 参考资料

1. [Android - FFmpeg & Mac & AndroidStudio & CMake 环境搭建](http://gavinliu.cn/2017/03/05/Android-FFmpeg-Mac-AndroidStudio-CMake-环境搭建/)
2. [FFmpeg-3.3.1移植到Android平台(Mac编译)](https://www.jianshu.com/p/da64059799d5)
3. [Mac 环境下ffmpeg编译出现 errno.h: No such file or directory 错误问题](https://blog.csdn.net/lakebobo/article/details/79607312)