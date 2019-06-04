---
title: ijkpalyer的官方指导编译
date: 2017-02-24 17:29:14
tags: [ijkpalyer,android]
---
ijkplayer是基于FFmpeg的开源的移动平台视频播放器，跨平台支持Android和IOS，支持本地播放和在线视频播放，有点类似google的开源播放器[ExoPlayer](https://github.com/google/ExoPlayer)，想从底层了解视频播放的过程，ijkplayer是一个非常值得学习的开源项目。
主要先介绍下ijkplayer的编译过程：

![ijkplayer-proccess](http://of6x0sb2r.bkt.clouddn.com/ijkplayer.png)

<!--more-->

# clone ijkplayer code source

```
git clone https://github.com/Bilibili/ijkplayer.git ijkplayer-android
```

![clone](http://of6x0sb2r.bkt.clouddn.com/clone%20ijkplayer.png-WaterMark)

# checkout latest branch

```
cd ijkplayer-android
git checkout -B latest k0.7.7.1
```

![clone](http://of6x0sb2r.bkt.clouddn.com/checkout-ijkplayer.png-WaterMark)

# init compile source

下载ffmpeg和libyuv的源码

从远程仓库下载编译所需要的源码，
```
./init-android.sh
```

![init](http://of6x0sb2r.bkt.clouddn.com/pull-ffmpeg.png-WaterMark)

![init](http://of6x0sb2r.bkt.clouddn.com/pull-libyuv.png-WaterMark)



# compile ffmepg

```
cd android/contrib
./compile-ffmepg.sh clean
./compile-ffmpeg.sh all
```

![compile ffmpeg](http://of6x0sb2r.bkt.clouddn.com/clean-ffmpeg.png-WaterMark)

![compile ffmpeg](http://of6x0sb2r.bkt.clouddn.com/check-archs-env-armv5.png-WaterMark)

![compile ffmepg](http://of6x0sb2r.bkt.clouddn.com/check-ffmpeg-env-configurate.png-WaterMark)

![compile ffmpeg](http://of6x0sb2r.bkt.clouddn.com/enable-configuration.png-WaterMark)

![compile-ffmpeg](http://of6x0sb2r.bkt.clouddn.com/compile-ffmpeg.png-WaterMark)

![compile-ffmpeg](http://of6x0sb2r.bkt.clouddn.com/link-ffmpeg-share-finish.png-WaterMark)

# compile ijkplayer

```
cd ..
./compile-ijk.sh
```

![compile-ijk](http://of6x0sb2r.bkt.clouddn.com/compile-ijk.png-WaterMark)

![compile-ijk](http://of6x0sb2r.bkt.clouddn.com/install-libijk-so.png-WaterMark)


# 编译完成打开官方工程

官方Android工程的文件夹在android/ijkplayer中，结构如下：
```
.
├── build
│   ├── generated
│   └── intermediates
├── build.gradle
├── gradle
│   └── wrapper
├── gradle.properties
├── gradlew
├── gradlew.bat
├── ijkplayer-arm64
│   ├── build
│   ├── build.gradle
│   ├── gradle.properties
│   ├── ijkplayer-arm64.iml
│   ├── proguard-rules.pro
│   └── src
├── ijkplayer-armv5
│   ├── build
│   ├── build.gradle
│   ├── gradle.properties
│   ├── ijkplayer-armv5.iml
│   ├── proguard-rules.pro
│   └── src
├── ijkplayer-armv7a
│   ├── build
│   ├── build.gradle
│   ├── gradle.properties
│   ├── ijkplayer-armv7a.iml
│   ├── proguard-rules.pro
│   └── src
├── ijkplayer-example
│   ├── build
│   ├── build.gradle
│   ├── ijkplayer-example.iml
│   ├── proguard-rules.pro
│   └── src
├── ijkplayer-exo
│   ├── build
│   ├── build.gradle
│   ├── gradle.properties
│   ├── ijkplayer-exo.iml
│   ├── proguard-rules.pro
│   └── src
├── ijkplayer.iml
├── ijkplayer-java
│   ├── build
│   ├── build.gradle
│   ├── gradle.properties
│   ├── ijkplayer-java.iml
│   ├── proguard-rules.pro
│   └── src
├── ijkplayer-x86
│   ├── build
│   ├── build.gradle
│   ├── gradle.properties
│   ├── ijkplayer-x86.iml
│   ├── proguard-rules.pro
│   └── src
├── ijkplayer-x86_64
│   ├── build
│   ├── build.gradle
│   ├── gradle.properties
│   ├── ijkplayer-x86_64.iml
│   ├── proguard-rules.pro
│   └── src
├── local.properties
├── settings.gradle
└── tools
    ├── gradle-bintray-upload.gradle
    ├── gradle-mvn-push.gradle
    └── gradle-on-demand.gradle

30 directories, 41 files

```

Android的官方Demo在ijkplayer-example中，后面文章会详细一点分析ijkplayer的编译过程和调用过程