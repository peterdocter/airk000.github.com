---
layout: post
title: "Android Gradle插件0.14.0新特性"
description: ""
category: "Android开发便签"
tags: [gradle]
---

随着Android Studio 0.9.0 Release，Android Gradle 插件的0.14.0版本也随之发布了，新特性是可以自动移除未使用的资源。而这个特性不仅限于应用自身，还能够移除所使用的library中未使用的资源。

###使用

这个特性默认是关闭的，想要使用它需要修改build.gradle中的配置。

    android {
        buildTypes {
            release {
                minifyEnabled true
                shrinkResources true
            }
        }
    }
