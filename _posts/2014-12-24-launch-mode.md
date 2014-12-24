---
layout: post
title: "Activity LaunchMode"
description: "Android应用中Activity的四种launchMode说明"
category: "Android开发便签"
tags: [Android]
---

Activity LaunchMode
===

应用程序的每一个Activity必须在AndroidManifest.xml中进行声明，而声明的时候开发者有机会指定该Activity的启动模式，包括四种：`standard`、`singleTop`、`singleTask`、`singleInstance`。

###Task的概念

Android系统使用Task管理Activity，也可以理解为Activity的管理栈。Task使用LIFO管理Activity，最后启动的Activity总是在栈顶的。Activity可以通过`getTaskId()`方法获取所在Task的ID，不过也就没有其他的方法接触Task了。

###Standard

不用特殊配置，系统默认。

特性是Android系统总会为目标Activity创建一个新的实例，然后将这个Activity放入当前的Task中，如果该Activity是当前应用的第一个Activity，则会创建Task来装载它。

###SingleTop

当被启动的Activity已经处于所在Task的栈顶时，Android系统不会创建新的Activity实例，效果上也就是什么也不做。而如果启动的Activity并没有处于Task的栈顶中（即使此时Activity已经存在于Task栈中而非栈顶而已），那么系统会创建一个新的Activity实例，将其放入Task栈顶。

###SingleTask

使用这种LaunchMode的Activity在一个Task栈中只能存在一个实例：

- 如果要启动的Activity不存在，系统将创建Activity实例，将其加入到所属Task的栈顶
- 如果要启动的Activity已经存在于栈顶，那么Android系统什么也不错，和singleTop一样
- 如果要启动的Activity已经存在，但是却不再所在Task的栈顶，那么Android系统会将其所在Task中所有在其上的Activity全部移除，使该Activity来到栈顶

###SingleInstance

使用这种LaunchMode的Activity无论从哪个Task中被启动，都会创建一个新的Task，然后创建一个新的Activity实例放入其中。并且，这个新的Task总是只包含一个Activity。

- 如果要启动的Activity不存在，那么系统会先创建一个新的Task，然后再创建新的Activity实例，将其放入其中
- 如果要启动的Activity已经存在，也就是说承载它的Task已经存在，Android系统会将此Task转到前台，此Activity就能展示在用户眼前了
