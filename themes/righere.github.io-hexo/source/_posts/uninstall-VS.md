---
title: 卸载visual studio 2013、2015等，微软打脸出品
date: 2017-02-15 14:08:00
tags: VisualStudio
---
最近下载了VS2015，想卸载之前安装的VS2013，却发现根本就发发卸载干净，想想安装一个版本的VS就需要消耗10个多G的磁盘空间，感觉好亏，必须的把它干掉，不错到微软怎么想的，无法卸载他们的开发软件是一种怎样的心理，难不成希望我么重装系统么？最好是重装成win10的系统么？再想想，微软为了推广自己的win10，下了好大的一盘棋-_-||
<!--more-->

就不多废话了，终于找了卸载VS2013以后版本的卸载神器了，github地址：https://github.com/Microsoft/VisualStudioUninstaller

一看居然是微软自己出品！！也是醉了，微软你这不是瞎折腾么？？找了这么长时间，之前还用命令符卸载：`vs_community.exe /uninstall /force`，发现白忙活了，还是卸载不干净。

![tool](http://of6x0sb2r.bkt.clouddn.com/uninstall_tool.png "工具截图")

以管理员方式运行强制卸载工具，_提醒一下这个软件会卸载vs2012及以前版本的更新_，

![uninstall](http://of6x0sb2r.bkt.clouddn.com/uninstallVitualStudio.png "卸载截图")

卸载之后终于清爽了，可以安装我们需要的VS版本了