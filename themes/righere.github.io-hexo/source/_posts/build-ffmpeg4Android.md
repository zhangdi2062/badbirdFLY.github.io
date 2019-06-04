---
title: 第一次完成FFmepg的移植，编译ffmpeg4Android
date: 2016-10-10 23:35:12
tags: [ffmpeg, android, 移植, 编译]
---

本文主要实现了FFmpeg的编译和移植，首先在linux下将官网下载的源码编译成.so文件，然后使用android-studio配合NDK工具，将.so文件移植到android项目当中，简单地介绍了如何一步步完成FFmpeg的编译流程

<!--more-->

参考文章：[手把手图文并茂教你用Android Studio编译FFmpeg库并移植](http://blog.csdn.net/hejjunlin/article/details/52661331)

下面是我自己在ubuntu下编译

## 准备的编译工具

Git，NDK

安装git，检查本地git，`git --version`

直接用命令符安装： `sudo apt install git`

也可以去官网下载，[git官网下载](https://git-scm.com/downloads)

如果已经有了git，想更新到最新版，可以输入 `git clone https://github.com/git/git`，再进行编译安装


然后下载NDK(现在已经有13的版本了)，推荐使用android studio安装ndk，下载的ndk路径默认在AndroidSDK的ndk-boudle文件中

## 配置NDK的环境
使用terminal配置电脑的环境,(个人电脑，我直接以管理员权限配置的系统环境变量)
> sudo gedit /etc/profile

打开之后，把我们的ndk路径配置进来，
```
	#Android NDK
	export ANDROID_NDK=/path/to/ndk(你的ndk解压路径)
	export PATH=$PATH:$ANDROID_NDK
```

## 下载ffmpeg的最新的源码
新建一个ffempeg的工作文件夹，例如 `mkdir workplace`
> git clone https://git.ffmpeg.org/ffmpeg.git

也可以直接去官网下载：[ffmpeg官网下载](https://ffmpeg.org/download.html)

## 修改ffmpeg的configure文件
为了保证我们编译出的文件是以.so的后缀名
将文本中的3209-3212这4行：
```
SLIBNAME_WITH_MAJOR='$(SLIBNAME).$(LIBMAJOR)'
LIB_INSTALL_EXTRA_CMD='$$(RANLIB) "$(LIBDIR)/$(LIBNAME)"'
SLIB_INSTALL_NAME='$(SLIBNAME_WITH_VERSION)'
SLIB_INSTALL_LINKS='$(SLIBNAME_WITH_MAJOR) $(SLIBNAME)'
```

修改改为：
```
SLIBNAME_WITH_MAJOR='$(SLIBPREF)$(FULLNAME)-$(LIBMAJOR)$(SLIBSUF)'
LIB_INSTALL_EXTRA_CMD='$$(RANLIB) "$(LIBDIR)/$(LIBNAME)"'
SLIB_INSTALL_NAME='$(SLIBNAME_WITH_MAJOR)'
SLIB_INSTALL_LINKS='$(SLIBNAME)'
```
## 编写编译脚本
写脚本的直观上的好处就是省去了一步步的执行编译命令，通常的在编译之前都需要进行配置，设置相应的环境变量，比如指定编译工具、编译平台等等，查看所有的配置选项可以在ffmpeg目录执行如下命令：
> ./configure --help



当然此脚本中，我们只需要设置几个我们比较关心的配置。

    1. NDK的路径：
        NDK=/path/to/ndk

    2. 编译的ndk plaform版本：
        SYSROOT=$NDK/platforms/android-23/arch-arm/

    3. 交叉编译工具:
        TOOLCHAIN=$NDK/toolchains/arm-linux-androideabi-4.9/prebuilt/linux-x86_64

    4. CPU平台：
        CPU=arm

    5. 编译完成后安装目录(当前目录下的android文件夹)：
        PREFIX=./android/$CPU

确定我们需要的配置之后，在ffmpeg的目录下面新建一个脚本 `build4android.sh`

```
	#!/bin/sh
	NDK=/path/to/ndk
	SYSROOT=$NDK/platforms/android-23/arch-arm/
	TOOLCHAIN=$NDK/toolchains/arm-linux-androideabi-4.9/prebuilt/linux-x86_64
	CPU=arm
	PREFIX=./android/$CPU
	ADDI_CFLAGS="-marm"
	config_para()
	{
	./configure \
		--prefix=$PREFIX \
		--enable-shared \
		--disable-static \
		--disable-doc \
		--disable-ffmpeg \
		--disable-ffplay \
		--disable-ffprobe \
		--disable-ffserver \
		--disable-avdevice \
		--disable-doc \
		--disable-symver \
		--cross-prefix=$TOOLCHAIN/bin/arm-linux-androideabi- \
		--target-os=linux \
		--arch=arm \
		--enable-cross-compile \
		--sysroot=$SYSROOT \
		--extra-cflags="-Os -fpic $ADDI_CFLAGS" \
		--extra-ldflags="$ADDI_LDFLAGS" \
		$ADDITIONAL_CONFIGURE_FLAG
	make clean
	make
	make install
	}
	config_para
```

这里增加一个方便的脚本技巧，便于修改自己的配置，我们将这些configure配置参数写进一个脚本function，建议如下语法：
```
func_name()
{
    ./configure \
    <---我们的参数配置---/>
}
```
另外一种一种语法（这种可能会报找不到function的错误，不推荐使用）：
```
function func_name
{
    ./configure \
    <---我们的参数配置---/>
}
```
还有就是脚本编辑推荐使用vim非常方便，变量参数一目了然
<center>
![FFmpegBuildScript](http://of6x0sb2r.bkt.clouddn.com/ffmpegBuildScript.png-WaterMark)
</center>

## 开始编译
进入我们的FFmpeg的源码目录，执行刚才的脚本：
（先增加执行权限 `sudo chmod +x build4android.sh`,以防权限不够）
> ./build4anroid.sh

脚本执行中：![脚本执行中图片](http://of6x0sb2r.bkt.clouddn.com/buildffmpeg_1.png-WaterMark)

脚本执行完成:![脚本执行完成图图片](http://of6x0sb2r.bkt.clouddn.com/finishffmpeg.png-WaterMark)

生成的`android/arm/`目录中的文件,图中标出的为链接文件可以删除：

![生成的.so文件图片](http://of6x0sb2r.bkt.clouddn.com/ffmpeglib.png-WaterMark)


## 移植我们的.so文件

### 拷贝.so文件到android-studio工程文件中

在android-studio中切换成project工程目录模式，在`app/src/main`目录下面新建一个`jni`文件夹，将上面第6步中的`include`文件夹拷贝到此文件夹中，再在`jni`文件夹中新建一个`lib`文件夹，将`.so`文件拷贝进来,新建`Android.mk`,`Application.mk`和c文件目录结构如下：

```
├── Android.mk
├── Application.mk
├── ffmpeg-jni.c
├── include
│   ├── libavcodec
│   ├── libavfilter
│   ├── libavformat
│   ├── libavutil
│   ├── libswresample
│   └── libswscale
└── lib
    ├── libavcodec-57.so
    ├── libavfilter-6.so
    ├── libavformat-57.so
    ├── libavutil-55.so
    ├── libswresample-2.so
    └── libswscale-4.so
```

### 编写Android.mk文件

```
    
    LOCAL_PATH := $(call my-dir)
    
    include $(CLEAR_VARS)
    LOCAL_MODULE:= libavcodec
    LOCAL_SRC_FILES:= lib/libavcodec-57.so
    LOCAL_EXPORT_C_INCLUDES:= $(LOCAL_PATH)/include
    include $(PREBUILT_SHARED_LIBRARY)
    
    include $(CLEAR_VARS)
    LOCAL_MODULE:= libavformat
    LOCAL_SRC_FILES:= lib/libavformat-57.so
    LOCAL_EXPORT_C_INCLUDES:= $(LOCAL_PATH)/include
    include $(PREBUILT_SHARED_LIBRARY)
    
    include $(CLEAR_VARS)
    LOCAL_MODULE:= libswscale
    LOCAL_SRC_FILES:= lib/libswscale-4.so
    LOCAL_EXPORT_C_INCLUDES:= $(LOCAL_PATH)/include
    include $(PREBUILT_SHARED_LIBRARY)
    
    include $(CLEAR_VARS)
    LOCAL_MODULE:= libavutil
    LOCAL_SRC_FILES:= lib/libavutil-55.so
    LOCAL_EXPORT_C_INCLUDES:= $(LOCAL_PATH)/include
    include $(PREBUILT_SHARED_LIBRARY)
    
    include $(CLEAR_VARS)
    LOCAL_MODULE:= libavfilter
    LOCAL_SRC_FILES:= lib/libavfilter-6.so
    LOCAL_EXPORT_C_INCLUDES:= $(LOCAL_PATH)/include
    include $(PREBUILT_SHARED_LIBRARY)
    
    include $(CLEAR_VARS)
    LOCAL_MODULE:= libswresample
    LOCAL_SRC_FILES:= lib/libswresample-2.so
    LOCAL_EXPORT_C_INCLUDES:= $(LOCAL_PATH)/include
    include $(PREBUILT_SHARED_LIBRARY)
    
    include $(CLEAR_VARS)
    LOCAL_MODULE:= FFmpegCodec
    LOCAL_SRC_FILES:= ffmpeg-jni.c
    LOCAL_C_INCLUDES += $(LOCAL_PATH)/include
    LOCAL_LDLIBS := -llog  -lz
    LOCAL_SHARED_LIBRARIES := avcodec avfilter avformat avutil  swresample swscale
    include $(BUILD_SHARED_LIBRARY)
```

### 编写Application.mk文件

```
    APP_ABI :=armeabi  //这里可选armeabi-v7a，x86,mips和all等
    APP_PLATFORM := android-23
    APP_OPTIM := release //默认是relaese，也可以设置成debug
    APP_STL := gnustl_static //这里是可选，当你编写下面的C文件的时候这里就可以体现出作用，例如你要使用`#include <string>`这个库的时候，默认的system库是没有包含这个的
```
### 编写C文件

在上面编写的Android.mk中，我们制定了c文件的名字为`ffmpeg-jni.c`，所以c文件的与此保持一致，

函数申明语法：JNIEXPORT jstring Java_包名_activity名_函数名，包名中间的点号`.`全部变成下划线`_`

例如：我在包名`com.righere.ffmpegndkbuild`下面的`MainActivity`中要使用avcodecInfo这个函数，

`JNIEXPORT jstring Java_com_righere_ffmpegndkbuild_MainActivity_avcodecInfo`

```
#include <stdio.h>
#include "libavformat/avformat.h"
#include <libavfilter/avfilter.h>
#include <jni.h>

JNIEXPORT jstring Java_com_righere_ffmpegndkbuild_MainActivity_avcodecInfo(JNIEnv* env, jobject obj)
{
    char info[4000] = { 0 };
    int count = 100;  //输出前100个codec名字
    
    av_register_all();//初始化所有decoder和encoder,注册所有容器类型和codec
    
    AVCodec *c_temp = av_codec_next(NULL);

    while (c_temp != NULL && count > 0){
        //输出解码器和编码器
        if(c_temp->decode != NULL){
            sprintf(info,"%s[Dec]",info);
        }
        else{
            sprintf(info,"%s[Enc]",info);
        }
        
        sprintf(info,"%s[%10s]\n",info,c_temp->name);
        
        c_temp = c_temp->next;
        count--;
    }
    return (*env)->NewStringUTF(env, info);
 }
```

### 修改gradle(修改的是app的`build.gradle`)

-   告诉gradle，`.mk`文件的路径(在android{--}里面添加)：

```
    externalNativeBuild {
        ndkBuild {
            path 'src/main/jni/Android.mk'
        }
    }
```
-   告诉gradle，完成编译之后，我们的生成的`.so`文件在哪个目录下面(在android{--}里面添加)：

这里我编译的`armeabi`版本的生成的`.so`文件自动会放在了'src/main/libs/armeabi'这个目录下面，其他的版本只需要把`armeabi`名字变一下就行了
```
    sourceSets.main {
        jni.srcDirs = [] //disable automatic ndk-build
        jniLibs.srcDirs = ['src/main/libs/armeabi']
    }
```
-   告诉gradle，ndk-build的时候还要执行哪些附加选项(在defaultConfig{}里面添加):

这一步其实可以看做是对`.mk`文件的补充

```
    ndk {
            abiFilter "armeabi"     //在Application.mk里面设置保持一致，如果是编译了多个平台的，可以指定编译多平台的或者
            moduleName "FFmpegCodec"    //jni模块的名字，与android.mk文件保持一致
            ldLibs "log", "z", "m", "jnigraphics", "android"    //这里可以设置ldLibs选项，比如你想调试jni的时候必须添加"log"参数

        }
```
### 执行ndk-build

<center>
![ndk-build](http://of6x0sb2r.bkt.clouddn.com/ndk-build.png-WaterMark)
</center>

编译完成main文件夹下会增加`libs`和`obj`两个文件夹，其中`libs/armeabi`下就是我们需要的`.so`文件
```
├── jni
│   ├── Android.mk
│   ├── Application.mk
│   ├── ffmpeg-jni.c
│   ├── include
│   └── lib
├── libs
│   └── armeabi
├── obj
│   └── local
└── res
```

切换回Android结构的工程试图，系统会自动把生成的`.so`文件放入`jniLibs`中，看到这个FFmpeg的移植基本就成功了
<center>
![Android—structure](http://of6x0sb2r.bkt.clouddn.com/jniLibs.png-WaterMark)
</center>

### 测试FFmpeg移植

在MainActivity中添加测试代码，

```
public class MainActivity extends AppCompatActivity {
    //jni
   public native String avcodecInfo();

    static {
        System.loadLibrary("FFmpegCodec");
        System.loadLibrary("avcodec-57");
        System.loadLibrary("avfilter-6");
        System.loadLibrary("avformat-57");
        System.loadLibrary("avutil-55");
        System.loadLibrary("swresample-2");
        System.loadLibrary("swscale-4");
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        TextView Codec_info = (TextView) findViewById(R.id.TextView_codec_info);
        Codec_info.setMovementMethod(ScrollingMovementMethod.getInstance());
        Codec_info.setText(avcodecInfo());
    }
}
```