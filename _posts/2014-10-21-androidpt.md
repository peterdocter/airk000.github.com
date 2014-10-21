---
layout: post
title: "pt单位在开发中的应用"
description: "pt to sp"
category: "Android开发便签"
tags: [Android]
---

###pt

在Android应用设计中，pt这个单位经常出现，用来描述字体大小。其为point的缩写，是一个标准的长度单位，1pt＝1/72英寸，代表一个点。

###开发中使用

Android官方建议字体的大小在开发中使用sp为单位，sp是scaled pixels放大像素，能够最大限度的保证在不同分辨率、密度的设备上文字有最符合设计原稿的效果。当设计中字体的单位为pt时，就需要进行转换：

	1pt≈2.22sp

转换过后，将sp设置在xml中，最终的效果基本与设计无异。
