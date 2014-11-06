---
layout: post
title: "多状态Drawable和Color的定义"
description: "Android开发过程中经常要对控件自定义背景、颜色等，怎样不丢失状态"
category: "Android开发便签"
tags: []
---

Android应用开发过程中，经常要对很多控件进行自定义，其中必不可少的就是定义其背景、颜色（包括其内部显示的文字颜色），往往定义会因为缺少个别Android内置的控件状态而导致效果不尽人意。下边从Android源码中找出对Drawable自定义和对Color自定义的selector作为参考：

###Drawable(res/drawable)


	<selector xmlns:android="http://schemas.android.com/apk/res/android">
    	<item android:state_window_focused="false" android:state_enabled="true"
        	android:drawable="@drawable/btn_default_normal_holo_light" />
    	<item android:state_window_focused="false" android:state_enabled="false"
        	android:drawable="@drawable/btn_default_disabled_holo_light" />
    	<item android:state_pressed="true"
        	android:drawable="@drawable/btn_default_pressed_holo_light" />
    	<item android:state_focused="true" android:state_enabled="true"
        	android:drawable="@drawable/btn_default_focused_holo_light" />
    	<item android:state_enabled="true"
        	android:drawable="@drawable/btn_default_normal_holo_light" />
    	<item android:state_focused="true"
        	android:drawable="@drawable/btn_default_disabled_focused_holo_light" />
    	<item
         	android:drawable="@drawable/btn_default_disabled_holo_light" />
	</selector>


###Color(res/color)

一般很少用到Color的定义，算是冷门了：


	<selector xmlns:android="http://schemas.android.com/apk/res/android">
    	<item android:state_enabled="false" android:color="@android:color/bright_foreground_dark_disabled"/>
    	<item android:state_window_focused="false" android:color="@android:color/bright_foreground_dark"/>
    	<item android:state_pressed="true" android:color="@android:color/bright_foreground_dark_inverse"/>
    	<item android:state_selected="true" android:color="@android:color/bright_foreground_dark_inverse"/>
    	<item android:state_activated="true" android:color="@android:color/bright_foreground_dark_inverse"/>
    	<item android:color="@android:color/bright_foreground_dark"/> <!-- not selected -->
	</selector>

###状态

selector中的状态更像是过滤器，状态从最上边层层过滤到最下边，被拦截到就返回对应的属性值。多个状态可以组合匹配。