---
title: 编译Rk3288源码
date: 2017-03-14 17:52:32
tags: Android Compile
---
记录一下编译RK3288的过程，编译参考[官方网站](http://wiki.t-firefly.com/index.php/Firefly-RK3288/Build_android_lollipop#.E4.B8.8B.E8.BD.BD_Android_SDK)和[官方社区](http://developer.t-firefly.com/forum-100-1.html)
<!--more-->
# 获取源码
![git reset](http://of6x0sb2r.bkt.clouddn.com/rk3288waitingfor-gitreset.png-WaterMark)

# 编译内核
![compie-kernel](http://of6x0sb2r.bkt.clouddn.com/rk3288make-defconfig.png-WaterMark)

![make-kernel](http://of6x0sb2r.bkt.clouddn.com/rk3288make-kernel.png-WaterMark)

![kernel-success](http://of6x0sb2r.bkt.clouddn.com/rk3288build-kernel-success.png-WaterMark)

# 编译系统

![build-sh](http://of6x0sb2r.bkt.clouddn.com/rk3288build-sh.png-WaterMark)

![make-error](http://of6x0sb2r.bkt.clouddn.com/rk3288make-error.png-WaterMark)

![change-jdk](http://of6x0sb2r.bkt.clouddn.com/rk3288change-jdk.png-WaterMark)

![make-success](http://of6x0sb2r.bkt.clouddn.com/rk3288make-success.png-WaterMark)

# 编译过程中的问题

## 错误详情1：
```
prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.11-4.6//x86_64-linux/bin/ld: error: out/host/linux-x86/obj32/SHARED_LIBRARIES/libnativehelper_intermediates/JNIHelp.o: unsupported reloc 43 against global symbol std::string::_Rep::_S_empty_rep_storage
prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.11-4.6//x86_64-linux/bin/ld: error: out/host/linux-x86/obj32/SHARED_LIBRARIES/libnativehelper_intermediates/JNIHelp.o: unsupported reloc 43 against global symbol std::string::_Rep::_S_empty_rep_storage
```
- 解决办法：
```
cd prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.11-4.6/x86_64-linux/bin/
mv ld ld.old
ln -s /usr/bin/ld.gold ld
```
## 错误详情2:
```
build/core/droiddoc.mk:158: recipe for target 'out/target/common/docs/system-api-stubs-timestamp' failed
```
- 解决办法：
修改droiddc.mk的194行
```
vim build/core/droiddoc.mk
```
修改之后如下：
```
##
##
## standard doclet only
##
##
$(full_target): $(full_src_files) $(full_java_lib_deps)
	@echo Docs javadoc: $(PRIVATE_OUT_DIR)
	##@mkdir -p $(dir $@)
	@mkdir -p $@
	$(call prepare-doc-source-list,$(PRIVATE_SRC_LIST_FILE),$(PRIVATE_JAVA_FILES), \
			$(PRIVATE_SOURCE_INTERMEDIATES_DIR) $(PRIVATE_ADDITIONAL_JAVA_DIR))
	$(hide) ( \
		javadoc \
                -encoding UTF-8 \
                $(PRIVATE_DROIDDOC_OPTIONS) \
                \@$(PRIVATE_SRC_LIST_FILE) \
                -J-Xmx1024m \
                -XDignore.symbol.file \
                $(PRIVATE_PROFILING_OPTIONS) \
                $(addprefix -classpath ,$(PRIVATE_CLASSPATH)) \
                $(addprefix -bootclasspath ,$(PRIVATE_BOOTCLASSPATH)) \
                -sourcepath $(PRIVATE_SOURCE_PATH)$(addprefix :,$(PRIVATE_CLASSPATH)) \
                -d $(PRIVATE_OUT_DIR) \
                -quiet \
        && touch -f $@ \
    ) || (rm -rf $(PRIVATE_OUT_DIR) $(PRIVATE_SRC_LIST_FILE); exit 45)
```

## 错误详情3

```
target thumb C++: hwcomposer.rk30board <= hardware/rockchip/hwcomposer/rk_hwcomposer.cpp
target thumb C++: hwcomposer.rk30board <= hardware/rockchip/hwcomposer/rk_hwc_com.cpp
arm-linux-androideabi-g++: error: update: No such file or directory
arm-linux-androideabi-g++: error: RKUpdateService_box.apk: No such file or directory
arm-linux-androideabi-g++: error: for: No such file or directory
arm-linux-androideabi-g++: error: fix: No such file or directory
arm-linux-androideabi-g++: error: crach: No such file or directory
arm-linux-androideabi-g++: error: when: No such file or directory
arm-linux-androideabi-g++: error: check: No such file or directory
arm-linux-androideabi-g++: error: update": No such file or directory
build/core/binary.mk:620: recipe for target 'out/target/product/rk3288_box/obj/SHARED_LIBRARIES/hwcomposer.rk30board_intermediates/rk_hwcomposer.o' failed
make: *** [out/target/product/rk3288_box/obj/SHARED_LIBRARIES/hwcomposer.rk30board_intermediates/rk_hwcomposer.o] Error 1
make: *** Waiting for unfinished jobs....

```
- 解决办法： 对`hardware/rockchip/hwcomposer`下的文件转换文本格式：`find . -type f -exec dos2unix {} \;`

