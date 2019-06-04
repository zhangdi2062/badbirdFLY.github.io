---
title: MediaInfo库的使用
date: 2017-03-16 16:28:33
tags: MediaInfo
---
# MediaInfo
[MediaInfo](https://mediaarea.net/zh-CN/MediaInfo)是一款专门用来分析音频和视频的文件编码和内容信息的开源软件，通过MediaInfo可以快捷明了的获取多媒体文件信息，支持多平台（windows、mac、linux等），我们平时常用的[K-Lite Codec Pack](https://www.codecguide.com/download_kl.htm)就集成MediaInfo的功能，相比`FFmpeg`,`MediaInfo`获取多媒体信息的方式更加快捷丰富。

---
<!--more-->
**MediaInfo获取的文件信息有**：

- 内容信息：标题，作者，专辑名，音轨号，日期，总时间……
- 视频：编码器，长宽比，帧频率，比特率……
- 音频：编码器，采样率，声道数，语言，比特率……
- 文本：语言和字幕
- 段落：段落数，列表

**支持的文件格式有**：
- 视频：MKV, OGM, AVI, DivX, WMV, QuickTime, Real, MPEG-1, MPEG-2, MPEG-4, DVD (VOB)...
- 编码器：DivX, XviD, MSMPEG4, ASP, H.264/AVC, H.265/HEVC, FFV1... 
- 音频：OGG, MP3, WAV, RA, AC3, DTS, AAC, M4A, AU, AIFF...
- 字幕：SRT, SSA, ASS, SAMI...

**主要特点**

- 支持众多视频和音频文件格式
- 多种查看方式：文本，表格，树形图，网页……
- 自定义查看方式
- 信息导出：文本，CSV，HTML……
- 三种发布版本：图形界面，命令行，DLL(动态链接库)
- 与Windows资源管理器整合：拖放，右键菜单
- 国际化：有多种界面语言供选择，使用的unicode字符集很容易实现本地化

---

# 编译
源码下载：https://mediaarea.net/zh-CN/MediaInfo/Download

编译生成库文件都很简单，下载的源码会有两个第三方库：`ZenLib`和`zlibstate`,以及源码的主角`MediaInfoLib`，在windows环境下用Visual Studio就可以打开相应版本的工程文件就可以进行编译了，如VS2015的工程文件在目录`libmediainfo_0.7.93_AllInclusive\MediaInfoLib\Project\MSVC2015`下面，编译完成可以生成相应的`MediaInfo.lib`和`MediaInfo.dll`两个文件，放在我们自己新建的工程文件就可以使用了，新建的工程文件需要将`MediaInfoDLL.h`的头文件放在工程当中，使用了`MediaInfoDLL`的命名空间，和声明了两个类：`MediaInfo`和`MediaInfoList`
# 基本使用说明

**声明对象：**
```
MediaInfo MI;
或者
MediaInfoList MIL;
```
**查看使用的MediaInfo版本：**
```
String MediaInfo = MI.Option(__T("Info_Version"), __T(";;")).c_str();
```
**查看MediaInfo所有参数说明**
```
String MediaInfo = MI.Option(__T("Info_Parameters")).c_str();
```
**查看所有解码器说明**
```
String MediaInfo = MI.Option(__T("Info_Codecs")).c_str();
```
**打开视频文件**

```
MI.Open(Url); 
```

**显示视频的所有基本信息**
```
MI.Option(__T("Complete"));
String MediaInfo = MI.Inform().c_str();
```
**查看所有信息**
```
MI.Option(__T("Complete"), __T("1"));
String MediaInfo = MI.Inform().c_str();
```
**查看自定义信息**
```
MI.Option(__T("Inform"), __T("General;FileSize=%FileSize%")); //文件大小
MI.Option(__T("Inform"), __T("Video;Format=%Format%")); //视频格式
MI.Option(__T("Inform"), __T("Audio;Format=%Format%")); //音频格式
To_Display += MI.Inform().c_str();
```
**关闭对象**
```
MI.Close();
```

例如一个视频文件的所有信息如下：
```
General
Complete name               : videoTest.mp4
Format                      : MPEG-4
Format profile              : Base Media
Codec ID                    : isom (isom/iso2/avc1/mp41)
File size                   : 908 KiB
Duration                    : 10 s 0 ms
Overall bit rate mode       : Variable
Overall bit rate            : 744 kb/s
Encoded date                : UTC 1970-01-01 00:00:00
Tagged date                 : UTC 1970-01-01 00:00:00
Writing application         : Lavf52.64.2

Video
ID                          : 1
Format                      : AVC
Format/Info                 : Advanced Video Codec
Format profile              : Baseline@L3
Format settings, CABAC      : No
Format settings, ReFrames   : 3 frames
Codec ID                    : avc1
Codec ID/Info               : Advanced Video Coding
Duration                    : 10 s 0 ms
Bit rate                    : 607 kb/s
Nominal bit rate            : 1 024 kb/s
Width                       : 480 pixels
Height                      : 360 pixels
Display aspect ratio        : 4:3
Frame rate mode             : Variable
Frame rate                  : 24.000 FPS
Minimum frame rate          : 15.000 FPS
Maximum frame rate          : 30.000 FPS
Color space                 : YUV
Chroma subsampling          : 4:2:0
Bit depth                   : 8 bits
Scan type                   : Progressive
Bits/(Pixel*Frame)          : 0.146
Stream size                 : 741 KiB (82%)
Writing library             : x264 core 85 Ubuntu_2:0.85.1448+git1a6d32-4
Encoding settings           : cabac=0 / ref=3 / deblock=1:0:0 / analyse=0x1:0x111 / me=hex / subme=7 / psy=1 / psy_rd=1.00:0.00 / mixed_ref=0 / me_range=16 / chroma_me=1 / trellis=0 / 8x8dct=0 / cqm=0 / deadzone=21,11 / fast_pskip=1 / chroma_qp_offset=-2 / threads=4 / sliced_threads=0 / nr=0 / decimate=1 / mbaff=0 / constrained_intra=0 / bframes=0 / wpredp=0 / keyint=250 / keyint_min=25 / scenecut=40 / intra_refresh=0 / rc_lookahead=40 / rc=abr / mbtree=1 / bitrate=1024 / ratetol=3.9 / qcomp=0.60 / qpmin=10 / qpmax=51 / qpstep=4 / ip_ratio=1.41 / aq=1:1.00
Encoded date                : UTC 1970-01-01 00:00:00
Tagged date                 : UTC 1970-01-01 00:00:00

Audio
ID                          : 2
Format                      : AAC
Format/Info                 : Advanced Audio Codec
Format profile              : LC
Codec ID                    : 40
Duration                    : 9 s 915 ms
Bit rate mode               : Variable
Bit rate                    : 132 kb/s
Channel(s)                  : 2 channels
Channel positions           : Front: L R
Sampling rate               : 44.1 kHz
Frame rate                  : 43.066 FPS (1024 spf)
Compression mode            : Lossy
Stream size                 : 160 KiB (18%)
Encoded date                : UTC 1970-01-01 00:00:00
Tagged date                 : UTC 1970-01-01 00:00:00
```
点此处可以下载已经写好的[MediaInfoDemo](https://github.com/righere/MediaInfoDemo.git)

  
