---
layout: post
title: "Hello, BeagleBone Black!"
description: ""
category: "BeagleBone Black"
tags: [BBB]
---

BeagleBone Black上手笔记
===

###开机

BeagleBone Black（简称BBB）提供两种供电方式：

- USB供电
- DC供电

开机很简单，只要将任意一种电源直接插上就可以了。等待电源灯另一边的user leds闪起，就启动成功了。

###连接

因为目前没有HDMI线，所以只能通过终端连接。关于驱动安装按照<http://beagleboard.org/Getting%20Started> 的Step1/2即可。

在Step3的时候会教你如何连接你的BBB，浏览器打开192.168.7.2即可，其实是运行了BBB文件目录的Getstart.html。不过如果能成功连接的话，就代表BBB的网络连接没有问题。BBB出厂的时候带的是Angstrom系统（对这个系统毫不熟悉），可以在终端中通过终端进行连接BBB：

    ssh root@192.168.7.2
    passwd: root


**上手后的工作基本就是连接一下，执行一些终端命令就好了**
