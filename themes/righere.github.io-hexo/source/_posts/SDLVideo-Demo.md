---
title: Android平台上使用SDL官方demo播放视频（使用ffmpeg解码）
date: 2017-02-21 16:41:33
tags: [ffmpeg, android, SDL]
---
> SDL（Simple Directmedia Layer）是一套开源的跨平台多媒体开发库，集成了音视频的许多底层的API，介绍Windows平台下的例子已经很多了，例如：雷老师的[ 最简单的视音频播放示例7：SDL2播放RGB/YUV](http://blog.csdn.net/leixiaohua1020/article/details/40525591),既然SDL是跨平台的，自己有参考了雷老师的这篇文章[ 最简单的基于FFmpeg的移动端例子附件：SDL Android HelloWorld](http://blog.csdn.net/leixiaohua1020/article/details/47059553),下面将介绍下Android平台上播放视频的简单的例子
<!--more-->

# SDL结构框图

SDL4Android中接口都是使用c语言写的，接口调用使用的是java，android调用c的接口需要用到jni（java native interface）技术,需要先将c接口编译成共享库`.so`文件，目前可以使用两种工具进行编译：ndk和cmake(android-studio2.2以后的版本才支持cmake编译)，下面的框图显示的是使用ndk工具编译SDL的接口：
![SDL源码框架forAndroid](http://of6x0sb2r.bkt.clouddn.com/SDL2.0_Android%20project%20Structure.png-WaterMark "SDL源码框架forAndroid")

SDL的文件源码主要分为两部分，一部分是java代码，主要包含有Android上层使用的SDLActivity上层接口调用函数；另一部分就是SDL-jni，包含所有SDL底层的源码、自定义的接口源码以及将这些源码编译成共享库所需的Android.mk文件。

如图中所示，`SDL_android.c`和`SDL_android_mian.c`是官方demo中编译官方接口的c源文件，其中`SDL_android.c`编译21个接口放在`libSDL2.so`共享库中；在此次使用SDL播放视频的Demo中，`SDL_android_mian.c`配合我们自定义的`Custom.c`(可以定义多个或者一个文件)只编译一个`nativeInit()`接口放在`libmain.so`共享库中

# 下载SDL源码

SDL的介绍就不多说了，详细可以访问[SDL官方网站](https://www.libsdl.org/index.php),推荐一个中文学习网站[SDL中文教程](http://kelvmiao.info/sdl-tutorial-cn/index.html),但是里面的API版本是1.2的，可以了解一下，还有就是外文的教程网站[Lazy Foo](http://lazyfoo.net/tutorials/SDL/index.php),利用SDL播放视频，推荐访问[CSDN雷老师的博客](http://blog.csdn.net/leixiaohua1020/article/details/38979615)。

截至2017年2月，SDL版本已经更新到2.05，点击[官方下载地址](https://www.libsdl.org/download-2.0.php)
![download](http://of6x0sb2r.bkt.clouddn.com/SDL_download.png-WaterMark)

github上也可以下载[github下载地址](https://github.com/SDL-mirror/SDL.git)

在android工程中我需要用到的源码,android-project、include、src三个文件夹

![sdl-android-source](http://of6x0sb2r.bkt.clouddn.com/sdl-android.png-WaterMark)

android-project中主要是官方的demo：jni文件夹中需要配置`.mk`编译文件，src文件夹中包含官方的android上层java代码；
include和src中就是SDL的底层C代码，编译之后方可被上层java调用。

# 合并ffmpeg和SDL源码

因为我们解码用到的是ffmepg，所以此次还要集成ffmpeg的源码，不了解ffmpeg的移植的话，可以查看我前面写的文章:[第一次完成FFmepg的移植，编译ffmpeg4Android](https://righere.github.io/2016/10/10/build-ffmpeg4Android/)

为了减少代码的层级，我把SDL的源码和FFmepg的源码混合了在一起了

![sdl-ffmpeg](http://of6x0sb2r.bkt.clouddn.com/SDL-ffmepg1.png-WaterMark "合并源码")

# 调用c函数的流程

SDL打开视频播放的过程实际上是创建一个`SDLActivity`的过程,`SDLActivity`会创建一个`SDLSurface`和一个layout布局,然后将`SDLSurface`添加到布局上面，视频画面就是在`SDLSurface`上显示出来，正如上图中，`SDLSurface`要调用nativeInit函数，必须先通过JNI实现图像解码显示功能

![调用流程](http://of6x0sb2r.bkt.clouddn.com/SDL4Android.png-WaterMark)

我们对`nativeInit`函数稍作修改，在`SDLActivity`中把视频的URI传给`nativeInt`函数，

`SDLActivity.java`声明中，
```
public static native int nativeInit(String string);
```
`SDL_android_main.c`中，
```
/* This prototype is needed to prevent a warning about the missing prototype for global function below */
JNIEXPORT int JNICALL Java_com_righere_convexdplayer_sdl_SDLActivity_nativeInit(JNIEnv* env, jclass cls, jstring string);

/* Start up the SDL app */
JNIEXPORT int JNICALL Java_com_righere_convexdplayer_sdl_SDLActivity_nativeInit(JNIEnv* env, jclass cls, jstring string)
{
    int i;
    int argc = 3;
    int status;
//    int len;
    char* argv[3];

    /* This interface could expand with ABI negotiation, callbacks, etc. */
    SDL_Android_Init(env, cls);

    SDL_SetMainReady();

    argv[0] = SDL_strdup("app_process");
    const char* utf = NULL;
    if(string){
        utf = (*env)->GetStringUTFChars(env, string, 0);
        if(utf){
            argv[1] = SDL_strdup(utf);
            (*env)->ReleaseStringUTFChars(env, string, utf);
        }
    }
    argv[2] = NULL;

    //argv[0] = "app_progress"  argv[1] = string  argv[2] = NULL

    /* Run the application. */

    status = SDL_main(argc, argv);

    /* Release the arguments. */

    for (i = 0; i < argc; i++) {

        SDL_free(argv[i]);
    }
    SDL_stack_free(argv);
    /* Do not issue an exit or the whole application will terminate instead of just the SDL thread */
    /* exit(status); */

    return status;
}
```

# 编写C源文件实现

先看下FFmpeg的解码和SDL的显示流程：
![ffmpeg和SDL](http://of6x0sb2r.bkt.clouddn.com/FFmpeg4Android.png-WaterMark)
```
//
// Created by righere on 16-11-22.
//
#include "stdio.h"

#ifdef __ANDROID__
#include <jni.h>
#include <android/log.h>
//使用NDK的log
#define LOGE(...) __android_log_print(ANDROID_LOG_ERROR,"ERROR: ", __VA_ARGS__)
#define LOGI(...) __android_log_print(ANDROID_LOG_INFO,"INFO: ", __VA_ARGS__)
#else
#define LOGE(format, ...) printf("ERROR: " format "\n",###__VA_ARGS__)
#define LOGI(format, ...) printf("INFO: " format "\n",###__VA_ARGS__)
#endif

#include <src/render/SDL_sysrender.h>
#include <src/video/SDL_sysvideo.h>
#include <libavutil/imgutils.h>
#include "SDL.h"


#include "libavcodec/avcodec.h"
#include "libavfilter/avfilter.h"
#include "libavformat/avformat.h"
#include "libswscale/swscale.h"

//屏幕参数
int SCREEN_W = 1920;
int SCREEN_H = 1080;

//设置buffer输出格式，YUV：1， RGB：0
#define BUFFER_FMT_YUV 0

int main(int argc, char** argv)
{

    //FFmpeg Parameters
    AVFormatContext *pFormatCtx;
    int             streamIdx;
    AVCodecContext  *pCodecCtx;
    AVCodecParameters *avCodecParameters;
    AVCodec         *pCodec;
    AVFrame         *pFrame, *pFrame_out;
    AVPacket        *packet;

    //size buffer
    uint8_t *out_buffer;

    static struct SwsContext *img_convert_ctx;

//SDL Parameters
    SDL_Window     *sdlWindow;
    SDL_Texture    *sdlTexture;
    SDL_Rect sdlRect;
    SDL_Renderer   *renderer;

    if(argc < 2)
    {
        LOGE("no media input!");
        return -1;
    }
    //获取文件名
    const char* mediaUri = (const char *) argv[1];

    av_register_all();//注册所有支持的文件格式以及编解码器

    //分配一个AVFormatContext
    pFormatCtx = avformat_alloc_context();

    //判断文件流是否能打开
    if(avformat_open_input(&pFormatCtx, mediaUri, NULL, NULL) != 0){
        LOGE("Couldn't open input stream! \n");
        return -1;
    }
    //判断能够找到文件流信息
    if(avformat_find_stream_info(pFormatCtx, NULL) < 0){
        LOGE("couldn't find open stream information !\n");
        return -1;
    }
    //打印文件信息
    av_dump_format(pFormatCtx, -1, mediaUri, 0);
    streamIdx=-1;
    for(int i=0; i<pFormatCtx->nb_streams; i++)
        //新版本ffmpeg将AVCodecContext *codec替换成*codecpar
        if(pFormatCtx->streams[i]->codecpar->codec_type==AVMEDIA_TYPE_VIDEO) {
            streamIdx=i;
            break;
        }
    if(streamIdx == -1){
        LOGE("Couldn't find a video stream !\n");
        return -1;
    }


    // Get a pointer to the codec context for the video stream
    avCodecParameters = pFormatCtx->streams[streamIdx]->codecpar;

    // Find the decoder for the video stream
    pCodec=avcodec_find_decoder(avCodecParameters->codec_id);


    if(pCodec){
        LOGI("find decoder: %d", avCodecParameters->codec_id);
    }

    //alloc a codecContext
    pCodecCtx = avcodec_alloc_context3(pCodec);


    //transform
    if(avcodec_parameters_to_context(pCodecCtx,avCodecParameters) < 0){
        LOGE("copy the codec parameters to context fail!");
        return -1;
    }
    //打开codec
    int errorCode = avcodec_open2(pCodecCtx, pCodec, NULL);
    if(errorCode < 0){
        LOGE("Unable to open codec!\n");
        return errorCode;
    };

    //alloc frame of ffmpeg decode
    pFrame = av_frame_alloc();

    if(pFrame == NULL){
        LOGE("Unable to allocate an AVFrame!\n");
        return -1;
    }

    //decode packet
    packet = av_packet_alloc();

    pFrame_out = av_frame_alloc();
#if BUFFER_FMT_YUV
    //output frame for SDL
    enum AVPixelFormat pixel_fmt = AV_PIX_FMT_YUV420P;

#else
    //output RGBFrame
    enum AVPixelFormat pixel_fmt = AV_PIX_FMT_RGBA;
#endif
    out_buffer = av_malloc((size_t) av_image_get_buffer_size(pixel_fmt,pCodecCtx->width,pCodecCtx->height,1));
    av_image_fill_arrays(pFrame_out->data,pFrame_out->linesize, out_buffer,
                         pixel_fmt,pCodecCtx->width,pCodecCtx->height,1);
    //transform size and format of CodecCtx,立方插值
    img_convert_ctx = sws_getContext(pCodecCtx->width, pCodecCtx->height, pCodecCtx->pix_fmt,
                                     pCodecCtx->width, pCodecCtx->height, pixel_fmt,
                                     SWS_BICUBIC, NULL,
                                     NULL, NULL);

    //初始化SDL
    if(SDL_Init(SDL_INIT_VIDEO | SDL_INIT_AUDIO | SDL_INIT_TIMER)) {
        LOGE("SDL_Init failed %s" ,SDL_GetError());
        exit(1);
    }

    //设置window属性
    sdlWindow = SDL_CreateWindow("Convexd_SDL",SDL_WINDOWPOS_UNDEFINED,SDL_WINDOWPOS_UNDEFINED,
                              SCREEN_W, SCREEN_H,SDL_WINDOW_RESIZABLE|SDL_WINDOW_FULLSCREEN|SDL_WINDOW_OPENGL);

    if(!sdlWindow) {
        LOGE("SDL: could not set video mode - exiting\n");
        exit(1);
    }
    //create renderer and set parameter
    renderer = SDL_CreateRenderer(sdlWindow,-1,0);

#if BUFFER_FMT_YUV
    Uint32 sdl_out_fmt = SDL_PIXELFORMAT_IYUV;
#else
    Uint32 sdl_out_fmt = SDL_PIXELFORMAT_RGBA32;
#endif
    //Allocate a place to put our yuv image on that screen,set sdl_display_resolution
    sdlTexture = SDL_CreateTexture(renderer,
                            sdl_out_fmt,SDL_TEXTUREACCESS_STREAMING,
                            pCodecCtx->width, pCodecCtx->height);
    //默认全屏可以不用设置
//    sdlRect.x = 0;
//    sdlRect.y = 0;
//    sdlRect.w = SCREEN_W;
//    sdlRect.h = SCREEN_H;

    // Read frames
    while (av_read_frame(pFormatCtx, packet) >= 0) {
        // Is this a packet from the video stream?
        if (packet->stream_index == streamIdx) {
            //decoder allocate frame to pFrame,new api
            LOGI("%s","Got Video Packet Succeed");
            int getPacketCode = avcodec_send_packet(pCodecCtx, packet);
            if(getPacketCode == 0) {
                int getFrameCode = avcodec_receive_frame(pCodecCtx, pFrame);
                LOGI("%d", getFrameCode);
                // Did we get a video frame?
                if (getFrameCode == 0) {
                    LOGI("%s", "Got Video Frame Succeed");

                    //scale Frame
                    sws_scale(img_convert_ctx, (const uint8_t *const *) pFrame->data,
                              pFrame->linesize, 0, pFrame->height,
                              pFrame_out->data, pFrame_out->linesize);
#if (BUFFER_FMT_YUV == 1)

                    SDL_UpdateYUVTexture(sdlTexture, NULL, pFrame_out->data[0], pFrame_out->linesize[0],
                                         pFrame_out->data[1],pFrame_out->linesize[1],
                                         pFrame_out->data[2],pFrame_out->linesize[2]);
#else
                     SDL_UpdateTexture(sdlTexture,NULL,pFrame_out->data[0],pCodecCtx->width*4);  //4通道：pitch = width×4
#endif

                    SDL_RenderClear(renderer);
                    SDL_RenderCopy(renderer, sdlTexture, NULL, NULL);
                    SDL_RenderPresent(renderer);

                    //Delay 40ms
                    SDL_Delay(20);
                }
                else if (getFrameCode == AVERROR(EAGAIN)) {
                    LOGE("%s", "Frame is not available right now,please try another input");
                }
                else if (getFrameCode == AVERROR_EOF) {
                    LOGE("%s", "the decoder has been fully flushed");
                }
                else if (getFrameCode == AVERROR(EINVAL)) {
                    LOGE("%s", "codec not opened, or it is an encoder");
                }
                else {
                    LOGI("%s", "legitimate decoding errors");
                }
            }
        }
        // Free the packet that was allocated by av_read_frame
        av_packet_unref(packet);
    }
    sws_freeContext(img_convert_ctx);
    av_frame_free(&pFrame_out);
    av_frame_free(&pFrame);
    avcodec_close(pCodecCtx);
    avformat_close_input(&pFormatCtx);
    SDL_Quit();
    return 0;
}
```

# 编写Android.mk文件

先新建一个自定义的c源文件`convexd_native_render.c`,我们的视频的解码和显示都会在这里面实现。
此工程中的`Android.mk`文件在移植ffmepg的`Android.mk`文件的基础上增加了编译`libSDL2main.so`和`libSDL2.so`的配置，如下：
```
###########################
#
# SDL static library
#
###########################

LOCAL_MODULE := SDL2_static

LOCAL_MODULE_FILENAME := libSDL2

LOCAL_SRC_FILES += $(LOCAL_PATH)/src/main/android/SDL_android_main.c

LOCAL_LDLIBS :=
LOCAL_EXPORT_LDLIBS := -Wl,--undefined=Java_com_righere_convexdplayer_sdl_SDLActivity_nativeInit -ldl -lGLESv1_CM -lGLESv2 -llog -landroid

include $(BUILD_STATIC_LIBRARY)

###########################
#
# SDL My Custom library
# add by ldq 20161122
#
###########################

include $(CLEAR_VARS)

LOCAL_MODULE := SDL2main

LOCAL_C_INCLUDES := $(LOCAL_PATH)/include \

LOCAL_EXPORT_C_INCLUDES := $(LOCAL_C_INCLUDES)

# Add your application source files here...

LOCAL_SRC_FILES := $(LOCAL_PATH)/src/main/android/SDL_android_main.c \
	$(LOCAL_PATH)/convexd_native_render.c

LOCAL_SHARED_LIBRARIES := SDL2 avcodec avfilter avformat avutil  swresample swscale

LOCAL_LDLIBS := -lGLESv1_CM -lGLESv2 -llog

include $(BUILD_SHARED_LIBRARY)
```
可以根据自己的路径配置mk文件

# 配置build.gradle

```
android{
    defaultConfig {
        ndk {
            abiFilter("armeabi")
            moduleName "SDL2main"
            ldLibs "log", "z", "m", "jnigraphics", "android"
        }
    }
    
    externalNativeBuild {
        ndkBuild {
            path 'src/main/jni/Android.mk'
        }
    }
}
    
```
# 实现播放
在手机根目录下新建一个`convexd`的文件夹，把播放视频放进去，打开我们打包的demo，选择视频就可以实现视频的播放功能。

[附上demo地址]：https://github.com/righere/ConvexdSDLPlayer.git