---
title: linux创建快捷方式
date: 2016-10-23 19:29:50
tags: linux
---

首先，linux每一个应用程序的快捷方式的配置文件都放置在`/usr/share/applications`这个文件夹下面，这些配置文件的后缀名都是.desktop,
我们创建一个应用程序的快捷方式，只需要仿照这些应用程序的配置写一个配置文件就行了。

<!--more-->

首先进入这个目录下，

> sudo su

> cd /usr/share/applications

> ls

![shortcut-list](http://of6x0sb2r.bkt.clouddn.com/linux-link.png "linux下面的快捷方式")

这里我自定义了android-studio2.2的应用程序的快捷方式，

    1. 先在其他的文件夹中新建一个快捷方式的配置文件，比如`AndroidStudio2.2.desktop`，

    2. 编辑这个配置文件，推荐使用vim编辑，因为在编辑代码的时候我们可以看到语法上是不是存在错误，
> vim AndroidStudio2.2.desktop

    ![shortcut-param](http://of6x0sb2r.bkt.clouddn.com/linuxlink_param.png "快捷方式配置")

    3. 配置文件参数简要

```
      [Desktop Entry]  //申明是快捷方式
      Name= Android-Studio  //快捷方式显示的名字（必选）
      Comment= Develop your Android Application  //文件方式说明（可选）
      Exec= /home/righere/Android/android-studio/bin/studio.sh %U  //快捷方式的指向的执行文件或者脚本，一定要注意最后的那个`%U` （必选）
      Icon= /home/righere/Android/android-studio/bin/studio.png   //快捷方式是的图标 （可选）
      Categories= Application;Development;IDE   //菜单栏中显示的类别（可选）
      version= 2.2.1  //软件版本号（可选）
      Type=Application  //快捷方式执行的是应用程序（必选）
      StartupNotify=true  //startup notify的一个协议，不懂（可选，当type为Application起作用）
      Terminal=false   //是否在terminal中运行 （可选，当type是Application时起作用）
```

    4. 对这个文件保存，并添加执行权限

> chmod a+x AndroidStudio2.2.desktop

这样我们就添加成功了快捷方式，

![img](http://of6x0sb2r.bkt.clouddn.com/shortcut.png "系统快捷方式")