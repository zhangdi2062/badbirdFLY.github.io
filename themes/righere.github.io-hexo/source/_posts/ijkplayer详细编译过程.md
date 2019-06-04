---
title: ijkplayer详细编译过程
date: 2017-04-26 09:50:02
tags: ijkplayer
---
接着之前的一篇简要的ijkplayer的编译过程，这一遍主要是详细描述ijkplayer编译的详细过程，跟着编译的脚本详细分析在ijkplayer从开源库的clone到完整地编译android共享库的过程。
<!--more-->
# 下载最新版的源码

```
git clone https://github.com/Bilibili/ijkplayer.git ijkplayer-android
cd ijkplayer-android
git checkout -B latest k0.7.9
```

# init-android.sh 下载编译所需的库

这一步会用到tools文件夹下面的工具脚本，  
## pull ffmpeg base
- `pull-repo-base.sh`,将`ffmpeg`源码`clone`到`extra/ffmpeg`文件夹，
```
sh pull-repo-bash.sh https://github.com/Bilibili/FFmpeg.git extra/ffmpeg
```
`pull-repo-base.sh`脚本
```
REMOTE_REPO=$1
LOCAL_WORKSPACE=$2


if [ -z $REMOTE_REPO -o -z $LOCAL_WORKSPACE ]; then
    echo "invalid call pull-repo.sh '$REMOTE_REPO' '$LOCAL_WORKSPACE'"
elif [ ! -d $LOCAL_WORKSPACE ]; then
    git clone $REMOTE_REPO $LOCAL_WORKSPACE
else
    cd $LOCAL_WORKSPACE
    git fetch --all --tags
    cd -
fi
```
简化一下，就是  
```
git clone https://github.com/Bilibili/FFmpeg.git extra/ffmpeg
```

## pull ffmpeg fork  
将上一步clone下来的ffmpeg源码，不同arm处理器架构的（armv5 armv7 arm64 x86 x86_64）的文件相应文件夹下面`android/contrib/`,
```
function pull_fork()
{
    echo "== pull ffmpeg fork $1 =="
    sh $TOOLS/pull-repo-ref.sh $IJK_FFMPEG_FORK android/contrib/ffmpeg-$1 ${IJK_FFMPEG_LOCAL_REPO}
    cd android/contrib/ffmpeg-$1
    git checkout ${IJK_FFMPEG_COMMIT} -B ijkplayer
    cd -
}

pull_fork "armv5"
pull_fork "armv7a"
pull_fork "arm64"
pull_fork "x86"
pull_fork "x86_64"
```
`pull-repo-ref.sh`脚本
```
REMOTE_REPO=$1
LOCAL_WORKSPACE=$2
REF_REPO=$3

if [ -z $1 -o -z $2 -o -z $3 ]; then
    echo "invalid call pull-repo.sh '$1' '$2' '$3'"
elif [ ! -d $LOCAL_WORKSPACE ]; then
    git clone --reference $REF_REPO $REMOTE_REPO $LOCAL_WORKSPACE
    cd $LOCAL_WORKSPACE
    git repack -a
else
    cd $LOCAL_WORKSPACE
    git fetch --all --tags
    cd -
fi
```
简化一下（以arm64为例），
```
git clone --reference extra/ffmpeg  https://github.com/Bilibili/FFmpeg.git android/contrib/ffmpeg-arm64
cd android/contrib/ffmpeg-arm64
git repack -a
```
`--reference`的作用就是会`clone`之前上一步的仓库缓存，达到节省不必要的`clone`时间，

## init-config.sh
```
cp config/module-lite.sh config/module.sh
```
这一步是配置编译ffmpeg的参数，配置文件我们可以自己编写，`config/`文件夹下面有已经预设的，`module-defualt.sh`是ffmpeg官方默认的编译参数，`module-lite.sh`包含通用的编译参数，`module-lite-hevc.sh`在通用的基础之上添加了hevc解码，也就是俗称的H265

## init-android-libyuv.sh
```
IJK_LIBYUV_UPSTREAM=https://github.com/Bilibili/libyuv.git
IJK_LIBYUV_FORK=https://github.com/Bilibili/libyuv.git
IJK_LIBYUV_COMMIT=ijk-r0.2.1-dev
IJK_LIBYUV_LOCAL_REPO=extra/libyuv

set -e
TOOLS=tools

echo "== pull libyuv base =="
sh $TOOLS/pull-repo-base.sh $IJK_LIBYUV_UPSTREAM $IJK_LIBYUV_LOCAL_REPO

echo "== pull libyuv fork =="
sh $TOOLS/pull-repo-ref.sh $IJK_LIBYUV_FORK ijkmedia/ijkyuv ${IJK_LIBYUV_LOCAL_REPO}
cd ijkmedia/ijkyuv
git checkout ${IJK_LIBYUV_COMMIT}
cd -
```
下载`libyuv`的库,简单来表述就是：
```
git clone https://github.com/Bilibili/libyuv.git extra/libyuv
```

# compile-ffmpeg.sh 编译ijkffmpeg.so共享库

首先清除源码中可能存在的编译缓存
```
cd android/contrib
./compile-ffmpeg.sh clean
./compile-ffmpeg.sh all
```
然后开始编译ffmpeg

`compile-ffmepg.sh`后面的参数代表编译哪个arm建构的版本，默认编译`armv7a`,编译选项，后面执行的`compile-ijk.sh`后面的选项也是同样的道理。

```
armv5 armv7a arm64 x86 x86_64(指定编译哪个版本)
all32（所有32位处理器版本，包含armv5 armv7a x86）
all （所有通用版本，包含armv5 armv7a arm64 x86 x86_64）
clean （清除之前编译的缓存）
check (检测支持的版本)
```
编译的过程中，  
## check archs
  确认编译的相应处理器架构版本
  ```
    echo "FF_ALL_ARCHS = $FF_ACT_ARCHS_ALL"
    echo "FF_ACT_ARCHS = $*"
  ```
  例如会汇编armv7a，输出的确认信息如下：
  ```
  FF_ALL_ARCHS = armv5 armv7a arm64 x86 x86_64
  FF_ACT_ARCHS = armv7a
  ```
## 执行`do-pcompile-ffmeg.sh`
开始执行编译，脚本路径：`android/contrib/tools/do-compile-ffmepg.sh`
以编译`arm64`为例，简要表述这一步，
```
sh tools/do-compile-ffmpeg.sh arm64
```
### `check env`
这一步会检测，我们执行编译脚本`compile-ijk.sh`时，有没有添加`all armv5 armv7a arm64 x86 x86_64`等选项,否则会提醒:`You must specific an architecture 'arm, armv7a, x86, ...`

### `make NDK standalone toolchain`  
这一步调用`tools/do-detect-env.sh`,脚本路径：`android/contrib/tools/do-detect-env.sh`  
检测编译环境，用于检测NDK是否在环境变量中存在，包括NDK中的gcc版本和arm-linux-androideabi的版本，
会添加一些编译的参数，这里我们会关注一个编译参数，`FF_CFG_FLAGS`:编译ffmpeg添加的额外的参数（config/module.sh中已经添加了一部分参数），`FF_PREFIX`编译之后输出的文件目录，例如arm64输出目录是：`android\contrib\build\ffmpeg-arm64\output`

### check ffmpeg env

这一步加载fffmpeg的配置文件,这个文件就是我们之前配置的init-config.sh中配置的文件
```
export COMMON_FF_CFG_FLAGS=. $FF_BUILD_ROOT/../../config/module.sh` 
```

### configurate ffmpeg
这一步是配置ffmepg的编译参数，脚本代码块如下：
```
./configure $FF_CFG_FLAGS \
        --extra-cflags="$FF_CFLAGS $FF_EXTRA_CFLAGS" \
        --extra-ldflags="$FF_DEP_LIBS $FF_EXTRA_LDFLAGS"
    make clean
```
`FF_CFG_FLAGS`就是前两步配置的编译参数。

### compile ffmpeg
这一步是编译正式开始编译ffmpeg,脚本代码如下：

```
cp config.* $FF_PREFIX
make $FF_MAKE_FLAGS > /dev/null
make install
mkdir -p $FF_PREFIX/include/libffmpeg
cp -f config.h $FF_PREFIX/include/libffmpeg/config.h
```
这一步之后就完成了FFmpeg的编译过程，`cp config.* $FF_PREFIX`这一字段，就是在编译之前，把我们的编译的配置信息log都被复制到了相应的moudle文件夹，路径：`android/contrib/build/ffmpeg-arm64/output/`
![config_fate](http://of6x0sb2r.bkt.clouddn.com/config_fate.png-WaterMark)
![config_h](http://of6x0sb2r.bkt.clouddn.com/config_h.png-WaterMark)
![config_log](http://of6x0sb2r.bkt.clouddn.com/config_log.png-WaterMark)
![config_mak](http://of6x0sb2r.bkt.clouddn.com/config_mak.png-WaterMark)

### link ffmpeg
关键的一步来了，正常情况下，我们编译出来的ffmpeg的包含有`libavcodec.a libavfilter.a libavformat.a libavutil.a libswresample.a libswscale.a `，这六个输出库，`ijkplayer`
将这六个库编译成一个独立`ijkplayer.so`的共享库,
```
$CC -lm -lz -shared --sysroot=$FF_SYSROOT -Wl,--no-undefined -Wl,-z,noexecstack $FF_EXTRA_LDFLAGS \
    -Wl,-soname,libijkffmpeg.so \
    $FF_C_OBJ_FILES \
    $FF_ASM_OBJ_FILES \
    $FF_DEP_LIBS \
    -o $FF_PREFIX/libijkffmpeg.so
```
以编译arm64版本为例，
```
aarch64-linux-android-gcc -lm -lz -shared --sysroot=/ijkplayer-android/android/contrib/build/ffmpeg-arm64/toolchain/sysroot -Wl,  
--no-undefined -Wl,-z,  
noexecstack -Wl,-soname,libijkffmpeg.so  
compat/*.o libavcodec/*.o libavfilter/*.o libavformat/*.o libavutil/*.o libswresample/*.o libswscale/*.o  
libavcodec/aarch64/*.o libavcodec/neon/*.o libavutil/aarch64/*.o libswresample/aarch64/*.o libswscale/aarch64/*.o  
-o /ijkplayer-android/android/contrib/build/ffmpeg-arm64/output/libijkffmpeg.so
```

### create files for shared ffmpeg

这一步将生成的库文件拷贝到相应的共享文件夹，
```
mysedi() {
    f=$1
    exp=$2
    n=`basename $f`
    cp $f /tmp/$n
    sed $exp /tmp/$n > $f
    rm /tmp/$n
}

echo ""
echo "--------------------"
echo "[*] create files for shared ffmpeg"
echo "--------------------"
rm -rf $FF_PREFIX/shared
mkdir -p $FF_PREFIX/shared/lib/pkgconfig
ln -s $FF_PREFIX/include $FF_PREFIX/shared/include
ln -s $FF_PREFIX/libijkffmpeg.so $FF_PREFIX/shared/lib/libijkffmpeg.so
cp $FF_PREFIX/lib/pkgconfig/*.pc $FF_PREFIX/shared/lib/pkgconfig
for f in $FF_PREFIX/lib/pkgconfig/*.pc; do
    # in case empty dir
    if [ ! -f $f ]; then
        continue
    fi
    cp $f $FF_PREFIX/shared/lib/pkgconfig
    f=$FF_PREFIX/shared/lib/pkgconfig/`basename $f`
    # OSX sed doesn't have in-place(-i)
    mysedi $f 's/\/output/\/output\/shared/g'
    mysedi $f 's/-lavcodec/-lijkffmpeg/g'
    mysedi $f 's/-lavfilter/-lijkffmpeg/g'
    mysedi $f 's/-lavformat/-lijkffmpeg/g'
    mysedi $f 's/-lavutil/-lijkffmpeg/g'
    mysedi $f 's/-lswresample/-lijkffmpeg/g'
    mysedi $f 's/-lswscale/-lijkffmpeg/g'
done
```
`mysedi()`函数中注意几个命令的使用方法：  
`bansename $f`表示去掉文件`f`的路径和后缀名;   
`sed`表示查找替换功能，`sed 's/\output/\/output\/shared/g'`是将`/output/`替换成`/output/shared/`  
这一步之后完成了对ffmpeg的编译，并且通过android的standalone-toolchain编译完成了ffmpeg的共享库-`libijkffmpeg.so`

# 开始编译ijkplayer.so共享库
```
cd ..
./compile-ijk.sh all
```
这一步进入`ijkplayer/`下相应版本的文件夹下面执行`ndk-build`命令，将打包编译生成`ijkplayer.so`,`ndk-build`命令将打包编译三部分的源码：  
`ffmepg`:之前打包编译的SRC_FILES:`ijkffmpeg.so`,`ffmepg`的`include`  
`android-ndk-prof`:编译`ijkprof`中的c源码，生成`android-ndk-profiler`的静态库，主要作用的打印`ndk-build`的log信息。  
`ijkmedia`:编译`ijkmedia/`下面的源码,其中主要包括了`ijk4a,jkplayer,ijksdl,ijksoundtouch,ijkyuv`这5个模块  
这一步完成之后，`ndk`根据上面列出的模块下面的`Android.mk`文件进行编译,最后生成`ijkplayer`的共享库

