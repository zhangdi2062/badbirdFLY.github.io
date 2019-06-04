---
title: CompileFFmpeg4Windows
date: 2017-03-14 17:43:57
tags: FFmpeg
---

最近项目需要想整一下windows下面的FFmpeg编译，当然可以直接下载[官方编译好的库](https://ffmpeg.zeranoe.com/builds/)，有个缺点就是官方的是默认的编译配置，所以如果我们想自定义配置ffmpeg的编译参数，还是得自己亲自来编译，官方提供三个版本的编译：
- static：只有编译完成的`exe`程序；
- shared：包含`dll`和`exe`;
- dev: 包含头文件`.h`,`lib`和`dll.a`
<!--more-->

先来几个参考网站，

1、[官方编译网站](https://trac.ffmpeg.org/wiki/CompilationGuide/WinRT)

2、[Microsoft官方实例](https://github.com/Microsoft/FFmpegInterop)

编译需要的原料：

1、FFmpeg源码 ----------------- [官方下载](https://ffmpeg.org/download.html)


2、MSYS2（编译环境） -------------[MSYS2官方下载](http://www.msys2.org/)

3、YASM ------------------------[官方下载地址](http://yasm.tortall.net/Download.html)

4、gas-preprocessor ------------[下载地址](https://github.com/FFmpeg/gas-preprocessor)

# 安装msys2

msys2类似于cygwin（可以在windows上配置linux环境）和mingw（git的bash环境），安装完成之后,先需要把安装目录下的`msys2_shell.cmd`中注释掉的`rem set MSYS2_PATH_TYPE=inherit`,改成`set MSYS2_PATH_TYPE=inherit`，主要是将vs的环境继承给`msys2`；接着打开msys2的shell，安装4个编译工具，
```
pacman -S make gcc perl diffutils
```
> 如果之前安装了cygwin，这里的安装可能会出现冲突问题，导致cygwin的bash窗口中没法使用一些命令，如：ls等等，一般会报错：
```
1 [main] ls (138392) D:\msys64\usr\bin\ls.exe: *** fatal error - cygheap base mismatch detected - 0x180305408/0x1802FE408.
This problem is probably due to using incompatible versions of the cygwin DLL.
Search for cygwin1.dll using the Windows Start->Find/Search facility
and delete all but the most recent version.  The most recent version *should*
reside in x:\cygwin\bin, where 'x' is the drive on which you have
installed the cygwin distribution.  Rebooting is also suggested if you
are unable to find another cygwin DLL.
```
这时候我们需要在我的电脑中改变一下环境变量，在Path中将`D:\cygwin64\bin`提到`D:\msys64\usr\bin`前面，没有的话可以先配置

# 替换yasm

将下载的`yasm-**-win64.exe`改成`yasm.exe`,替换msys安装目录`x:\msys64\usr\bin\yasm.exe`,可以做个备份把原来的改成`yasm.bak`

# 安装gas-preprocessor

将下载的gas-preprocessor.pl放到msys2安装目录下面`x:\msys64\usr\bin\gas-preprocessor.pl`

# 从VS的命令窗口重新启动msys2的bash窗口

这一步没有把握正确，编译很容易出错,例如`cl is unable to create an executable file.`正确的启动步骤是：先去打开VS的工具命令提示符，`C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Visual Studio 2015\Visual Studio Tools\Windows Desktop Command Prompts`，然后在命名窗口中使用命令符打开msys2的bash窗口，
我安装在D盘，依次执行命令如下：
```
D:
cd msys64
msys2_shell.cmd
```
在bash窗口中先检查，安装环境是否已经正确配置：

```
$ which cl
/d/Program Files (x86)/Microsoft Visual Studio 14.0/VC/BIN/amd64_x86/cl

$ which link
/d/Program Files (x86)/Microsoft Visual Studio 14.0/VC/BIN/amd64_x86/link

$ which yasm
/usr/bin/yasm

$ which cpp
/usr/bin/cpp

$ which gas-preprocessor.pl
/usr/bin/gas-preprocessor.pl
```
正确配置之后就可以进行编译了。

# 开始编译

编译ffmpeg，有两种方式，

##  微软官方Demo
可以按照[微软官方的Demo教程](https://github.com/Microsoft/FFmpegInterop)来编译，先使用git下载，
```
git clone git://github.com/microsoft/FFmpegInterop.git
cd FFmpegInterop
git clone git://source.ffmpeg.org/ffmpeg.git
```
下载完成之后，目录结构如下：
```
.
├── ARM
├── BuildFFmpeg.bat
├── Debug
├── FFmpegConfig.sh
├── FFmpegInterop
├── FFmpegInterop.pfx
├── FFmpegWin10.VC.db
├── FFmpegWin10.sln
├── FFmpegWin8.1.sln
├── LICENSE
├── README.md
├── Samples
├── Tests
├── UpgradeLog.htm
├── ffmpeg
└── ipch
```
直接在bash窗口中进入,执行执行编译相应版本的命令就行了
```
BuildFFmpeg.bat win10                     - Build for Windows 10 ARM, x64, and x86
BuildFFmpeg.bat phone8.1 ARM              - Build for Windows Phone 8.1 ARM only
BuildFFmpeg.bat win8.1 x86 x64            - Build for Windows 8.1 x86 and x64 only
BuildFFmpeg.bat phone8.1 win10 ARM        - Build for Windows 10 and Windows Phone 8.1 ARM only
BuildFFmpeg.bat win8.1 phone8.1 win10     - Build all architecture for all target platform
```
## 手动编译
首先步骤是`./configure `配置编译参数，可以参考微软Demo中的脚本`FFmpegConfig.sh`，先进入下载的FFmepg源码目录，
比如编译Win10的x64版本，
```
    ./configure \
        --toolchain=msvc \
        --disable-programs \
        --disable-d3d11va \
        --disable-dxva2 \
        --arch=x86_64 \
        --enable-shared \
        --enable-cross-compile \
        --target-os=win32 \
        --extra-cflags="-MD -DWINAPI_FAMILY=WINAPI_FAMILY_APP -D_WIN32_WINNT=0x0A00" \
        --extra-ldflags="-APPCONTAINER WindowsApp.lib" \
        --prefix=./Build/Windows10/x64
        make install
```
编译完成之后，ffmpeg的编译库就在`Build/Windows10/x64`目录下面,目录结构如下：
```
.
├── bin
│?? ├── avcodec-57.dll
│?? ├── avcodec.lib
│?? ├── avdevice-57.dll
│?? ├── avdevice.lib
│?? ├── avfilter-6.dll
│?? ├── avfilter.lib
│?? ├── avformat-57.dll
│?? ├── avformat.lib
│?? ├── avutil-55.dll
│?? ├── avutil.lib
│?? ├── swresample-2.dll
│?? ├── swresample.lib
│?? ├── swscale-4.dll
│?? └── swscale.lib
├── include
│?? ├── libavcodec
│?? ├── libavdevice
│?? ├── libavfilter
│?? ├── libavformat
│?? ├── libavutil
│?? ├── libswresample
│?? └── libswscale
└── lib
    ├── avcodec-57.def
    ├── avdevice-57.def
    ├── avfilter-6.def
    ├── avformat-57.def
    ├── avutil-55.def
    ├── pkgconfig
    ├── swresample-2.def
    └── swscale-4.def

11 directories, 21 files
```
