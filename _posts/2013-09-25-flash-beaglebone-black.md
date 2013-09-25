---
layout: post
title: "BeagleBone Black刷机"
description: ""
category: "BeagleBone Black"
tags: [BBB]
---
{% include JB/setup %}

BBB系统更新
====

###准备工作

- 一张4G以上的SDCard及读卡器
- 至少5V1A的DC电源（USB插口也可）
- 能够刷入的img包
- 7zip解压软件（http://www.7-zip.org/） Linux系统可以用xz命令解压 .xz文件
- Win32DiskImager （http://sourceforge.net/projects/win32diskimager/） Linux系统可以用usb-imagewritter


###寻找合适的系统更新

http://elinux.org/BeagleBoardUbuntu#eMMC:_BeagleBone_Black
> 我所采用的 
https://rcn-ee.net/deb/flasher/raring/BBB-eMMC-flasher-ubuntu-13.04-2013-08-24.img.xz

http://circuitco.com/support/index.php?title=Updating_The_Software
> 据老外说这个已经outdate了，不过很多东西还是很经典的

###制作SDCard

- 将下载下来的.img.xz包通过7zip或者Linux的xz命令进行解压


    xz -d xxx.img.xz
- 然后使用Win32DiskImager或usb-imagewritter将img包写入SDCard（当然，这样会抹掉SDCard的内容）

- SDCard制作完成后，将SDCard从读卡器中取出。


###更新系统

- 拔出BBB的所有外接线
- 将制作完成的SDCard插入BBB的SD卡槽
- 按住User button，同时插入电源，直到User LEDs开始闪烁
- 等待稍许，待User LEDs全部亮起，代表系统更新完成

![BeagleboneBlack](https://raw.github.com/airk000/airk000.github.com/master/images/CONN_REVA5A.jpg)


###常见问题

- 为什么插入SD卡后，按住User button，然后插入电源键没反映？
> 1. 可能是电源不符合；

> 2. 可能是SD卡没有正确烧写，要用读卡器；

> 3. Board broken!!!



