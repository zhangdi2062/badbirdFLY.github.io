---
title: Learning Android C++之Android.mk和Application.mk
date: 2016-10-27 21:53:25
tags: NDK
---
> `Android.mk`文件是`NDK`构建系统用来描述`C`和`C++`源文件，是轻量级的`Makefile`片段，在构建系统中会被一次或多次解析，帮助我们将`C`和`C++`源文件打包成模块，包括静态库和共享库等;要将`C\c++`编译成`.so`文件，除了`android.mk`文件，还需要`Application.mk`文件,`Application.mk`是用来描述应用程序需要的模块。可以将这两个`.mk`文件理解成`C\C++`源文件编译成`.so`库文件的配置文件。

---
<!--more-->
# Android.mk

- 构建系统利用LOCAL_PATH来定位源文件，并提供了一个my-dir的宏功能，通过该宏功能的返回值，放在当前目录下
  > LOCAL_PATH := $(call my-dir)

- CLEAR_VARS,清除除了LOCAL_PATH以外的LOCAL_<name>的变量
  > include $(CLEAR_VARS)

- 给模块设置一个唯一的名称，该模块会生成一个libname.so的名字
  > LOCAL_MODULE := name

- 指定用来构建和组装这个模块的源文件列表，多文件的之间用空格隔开
  > LOCAL_SRC_FILES := `*.cpp` `*.h` 

- 为了构建主程序能够使用的模块，必须将模块变成共享库
  > include $(BUILD_SHARED_LIBRARY)

- Android应用程序并不直接使用静态库，应用程序包中也不包含静态库，但是静态库可以用来构建共享库，一般第三方的源代码不是直接包含在原生项目中，而是将他们先编译成静态库，然后在并入共享库中
  > include $(BUILD_STATIC_LIBRARY)

- 使用其他NDK项目的共享模块，调用函数宏import-module，放在Android.mk文件的末尾,以免构建系统冲突
  > $(call import-module, path/to/shared-libname)

- 使用Prebuild库，特点：1. 不发布源代码的情况下共享自己的模块；2. 想使用预建版模块来加速构建过程
  > LOCAL_SRC_FILES := libname.so 
  
  > include $(PREBUILD_SHARED_LIBRARY)

- 构建可执行文件
  > include $(BUILD_EXECUTABLE)

- 构建系统定义的其他变量

|变量名|  说明|
|---|---|
|TARGET_ARCH|目标CPU的体系结构，例如：arm
|TARGET_PLATFORM|目标android平台的名称，例如：android-24
|TARGET_ARCH_ABI|目标CPU的体系结构和相应的abi名称，例如：armeambi-v7a
|TARGET_ABI|目标平台的名称和abi串联，例如：android-23-armeabi-v7a

- 模块说明部分的变量

|变量名|说明|
|---|---|
|LOCAL_MODULE_FILENAME|用来重新定义模块的输出名|
|LOCAL_CPP_EXTENSION|为C++源码添加其他拓展名，默认为.cpp|
|LOCAL_CPP_FEATURES|指明模块所依赖的具体的C++特性，例如：rtti|
|LOCAL_C_INCLUDE|NDK的安装目录为相对路径，用来搜索头文件|
|LOCAL_CFLGS|编译器标志，编译C/C++源文件时传给编译器|
|LOCAL_CPP_FLAGS|只编译c++源文件时传送给编译器|
|LOCAL_WHOLE_STATIC_LIBRARY|用来指明包含在所生成的共享库中的所有静态库|
|LOCAL_LDLIBS|用于传送要动态链接的系统库列表|
|LOCAL_ALLOW_UNDEFINDE_SYBOLS|用于禁止生成文件的过程中使用符号检查|
|LOCAL_ARM_MODE|指定要生成的ARM二进制类型，默认情况下使用16位指令生成，当该值被设置成arm就是指定使用32指令生成，构建文件添加`.arm`后缀名可以指定arm模式下构建指定文件|
|LOCAL_ARM_NEON|用来指定在源文件中应该使用arm的高级单指令多数据流（SIMD）内联函数，该值设置为true时，用.neon后缀名指定只构建带有NEON内联函数的特定文件|
|LOCAL_DISABLE_NO_EXECUTE|boolean类型，用来禁止NX Bit（永不执行）安全特性，该安全特性用来隔离应用程序代码区和存储区，以防止恶意软件通过植入存储区代码来控制软件|
|LOCAL_EXPORT_CFLAGS|记录一组编译器标志，如果其他模块通过变量LOCAL_STATIC_LIBRARIES或者LOCAL_SHARED_LIBRATIES使用本模块，该flag会被添加到这些其他模块的LOCAL_CFLAGS当中|
|LOCAL_EXPORT_CPPFLAGS|和LOCAL_EXPROT_CFLAGS一样，作用于c++代码上|
|LOCAL_EXPORT_LDFLAGS|和LOCAL_EXPORT_CFLAGS一样，作用于链接器上|
|LOCAL_EXPORT_C_INCLUDES|允许记录路径集，添加到LOCAL_C_INLCUDES当中|
|LOCAL_SHORT_COMMANDS|对于有大量资源或独立的共享库/静态库，应该设置为true|
|LOCAL_FILTER_ASM|过滤来自LOCAL_SEC_FILES的资源文件的应用程序|

# Application.mk

和Android.mk一样是放在jni文件夹下面，描述应用程序需要哪些模块，定义了所有模块的通用变量

|变量名|说明|
|---|---|
|APP_MODULES|声明构建的所有模块|
|APP_OPTIM|可以设置为release或者debug,改变生成二进制文件的优化级别|
|APP_CFLAGS|列出编译器标志，在编译时传给编译器|
|APP_CPPFLAGS|和APP_CLAGS相同，作用于C++源文件|
|APP_BUILD_SCRIPT|查找Android.mk构建文件|
|APP_ABI|默认为armeabi，可以添加mips，all等变量|
|APP_STL|NDK默认使用最小的stl库，该变量可以选择不同的STL实现|
|APP_GNUSTL_FORCE_CPP_FEATURES|表明所有模块都依赖于具体的C++特性，如RTTI,exceptions等|
|APP_SHORT_COMMANDS|大量源码时使用更短指令|