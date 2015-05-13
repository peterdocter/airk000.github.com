---
layout: post
title: "Selector's statelist"
description: "Selector的StateList深度认识"
category: "Android开发便签"
tags: [Android]
---


在Android开发过程中经常需要书写图像的selector XML配置文件，而这其中包含了众多的状态。下边就一一对其进行介绍。

>相关连接：https://developer.android.com/intl/zh-cn/guide/topics/resources/drawable-resource.html

###Selector的基本语法

```
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android"
    android:constantSize=["true" | "false"]
    android:dither=["true" | "false"]
    android:variablePadding=["true" | "false"] >
    <item
        android:drawable="@[package:]drawable/drawable_resource"
        android:state_pressed=["true" | "false"]
        android:state_focused=["true" | "false"]
        android:state_hovered=["true" | "false"]
        android:state_selected=["true" | "false"]
        android:state_checkable=["true" | "false"]
        android:state_checked=["true" | "false"]
        android:state_enabled=["true" | "false"]
        android:state_activated=["true" | "false"]
        android:state_window_focused=["true" | "false"] />
</selector>
```

#####根属性

- `android:constantSize` 如果当状态发生改变时Drawable的大小要始终保持一致的话则设置为true，false的话就代表Drawable的大小跟随当前状态。默认为false
- `android:dither` 图像抖动使能开关，true代表当bitmap和屏幕的像素配置不同时候启用图像抖动，false则关闭图像抖动。默认为true，更多关于图像抖动 [请点击](http://en.wikipedia.org/wiki/Dither)
- `android:variablePadding` 顾名思义，动态边距。设置为true的话Drawable的边距将根据当前选中的状态决定，false的话说明边距始终保持一致（跟随所有状态中最大的那个）。如果选择使能这项特性，那么需要开发者自己来处理状态改变时的布局动作，默认是false

#####状态列表

- `android:drawable` 必需的属性，代表该Drawable的默认图片，如果当所有状态都没有被选中的话，总是要有个撑场的，它就是了。

**前方高能**
****

请注意！请注意！下边列出的一系列属性，是`区分`优先级的，最先列出的优先级最高。也就是说，当高优先级状态被选中后，其他的所有状态将全部被忽略，这是要非常注意的地方！！！

- `android:state_pressed` true代表当所作用的对象被`按压`时（如Button被点击），该状态被启用。false代表所作用对象尚未被`按压`时的状态
- `android:state_focused` true代表所作用对象拥有`输入焦点`时(如用户选中了一个文本输入框)该状态被启用。false代表默认状态，无焦点状态。focused可以更广泛的指焦点，并不完全是输入框的输入标点之类
- `android:state_hovered` **API14引入了这个状态**。true代表当`有光标悬浮`在作用对象时（类似PC的鼠标悬浮于某个控件处）启用改状态。false代表默认状态，非悬浮状态。通常情况下，这种状态与state_focused使用同一个drawable
- `android:state_selected` true是当作用对象被`选择`时启用的状态，false为非选中状态。被选中状态的使用通常是伴随着focused状态不够用，举个例子就是当一个列表拥有焦点，但是其中的项目要被选中的时候
- `android:state_checkable` true代表被作用对象是`可被选中`时的状态。false即为被作用对象不可被选中
- `android:state_checked` true代表被作用对象处于`被选中`的状态，false则是未被选中。checked状态针对的是接口Checkable
- `android:state_enabled` true代表当作用对象处于`可用`的状态，false即不可用。
- `android:state_activated` **API11引入了这个状态**true代表作用对象处于`活跃`的状态，进一步说是处于持续存在的被选中状态中，比如列表中高亮显示上一次选中的内容。false则代表非活跃的状态。关于这个状态，在AbsListVuew中对于内容item选中状态的处理有所提及：

	```
    private void updateOnScreenCheckedViews() {
        final int firstPos = mFirstPosition;
        final int count = getChildCount();
        final boolean useActivated = getContext().getApplicationInfo().targetSdkVersion
                >= android.os.Build.VERSION_CODES.HONEYCOMB;
        for (int i = 0; i < count; i++) {
            final View child = getChildAt(i);
            final int position = firstPos + i;

            if (child instanceof Checkable) {
                ((Checkable) child).setChecked(mCheckStates.get(position));
            } else if (useActivated) {
                child.setActivated(mCheckStates.get(position));
            }
        }
    }
	```
	可以看出，activated状态与checked状态平行，如果item声明了Checkable接口，则当更新内容item时将优先走Checkable所提供的接口，否则的话当应用的TargetAPI为11以上时，将设置item的activated状态。所以，显而易见的，checked状态比activated状态的优先级要高。
- `android:state_window_focused` true代表当所作用对象所处的应用的window拥有焦点时（也就是说应用处于`前台可视状态`）的状态。false则代表所处应用失去焦点后的状态。举个例子，加入你的应用有全局类型的Dialog对话框，其中的某个view使用selector，如果你需要区分所处应用是否处于前台而给view赋予不同的外观时，这个状态正是你需要的了。

###最后的提醒

请一定记住，以上说明的这些状态拥有优先级，所以在XML的书写过程中请注意他们的定义先后顺序，通常情况下，android:drawable（也就是默认状态）需要写在最后，前面也说过了，至少要有个撑场的嘛。