---
layout: post
title: "ListView单选、多选模式下的自定义布局"
description: "ListView在单选或多选模式下如何让系统为我们筹划选取策略。"
category: "Android开发便签"
tags: [Android]
---

###引言

Android界面开发中，经常有让ListView是多选或单选模式的需求，而伴随着这种需求，通常ListView中的项目布局也是需要我们进行自定义的。在自定义布局中内置一个CheckBox或者RadioButton作为选择的标识，那么怎么管理这些选择的状态？是自己来记录哪个position是否选择的状态吗？其实并不需要这样，Android系统其实已经留给了我们线索，告诉我们她可以帮我们处理这种情况。

###线索

所谓的线索就在ListView源码中，当ListView中的项目被点击，会触发ListView的`setupChild`函数，此函数中的这一段代码就是线索：

	if (mChoiceMode != CHOICE_MODE_NONE && mCheckStates != null) {
	            if (child instanceof Checkable) {
                                        ((Checkable) child).setChecked(mCheckStates.get(position));
            } else if (getContext().getApplicationInfo().targetSdkVersion
                    >= android.os.Build.VERSION_CODES.HONEYCOMB) {
                child.setActivated(mCheckStates.get(position));
            }
    }

当我们设置过ChoiceMode后切ChoiceMode不为NONE，ListView会自行对Child view进行处理，如果child view声明了Checkable接口则调用其setChecked，如果没有的话调用其setActivated接口（这个接口需要复写View的`dispatchSetActivated`方法）。这样看来通过对ListView中子布局的处理就可以让系统帮我们管理选择的状态了。

###以Checkable形式对子布局进行处理

Checkable的方式相比于dispatchSetActivated方式更加灵活，能够暴露出更多接口以便使用。所以这里以Checkable为例进行说明。复写一个RelativeLayout，代码：

	package com.ithooks.ms.android.view.widget;

	import android.content.Context;
	import android.util.AttributeSet;
	import android.view.View;
	import android.widget.Checkable;
	import android.widget.RelativeLayout;

	/**
 	* Created by kevin on 14/11/7.
 	*/
	public class CheckableRelativeLayout extends RelativeLayout implements Checkable {
    	View checkableView;

    public CheckableRelativeLayout(Context context) {
        super(context);
    }

    public CheckableRelativeLayout(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    @Override
    protected void onAttachedToWindow() {
        super.onAttachedToWindow();
        int count = getChildCount();
        for (int i = 0; i < count; i++) {
            View child = getChildAt(i);
            if (child instanceof Checkable) {
                checkableView = child;
                break;
            }
        }
    }

    @Override
    protected void onDetachedFromWindow() {
        super.onDetachedFromWindow();
        checkableView = null;
    }

    @Override
    public void setChecked(boolean checked) {
        if (checkableView != null) {
            ((Checkable) checkableView).setChecked(checked);
        }
    }

    @Override
    public boolean isChecked() {
        return checkableView != null && ((Checkable) checkableView).isChecked();
    }

    @Override
    public void toggle() {
        if (checkableView == null) {
            return;
        }
        Checkable c = (Checkable) checkableView;
        c.setChecked(!c.isChecked());
    }
    }

这样一个Checkable的布局就写好了，如果布局中存在如CheckBox、RadioButton之类的控件，那么ListView的选择状态就会传递到他们的身上了。

###使用举例

定义这样一个ListView中的子布局：

	<?xml version="1.0" encoding="utf-8"?>
	<com.xxx.widget.CheckableRelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <ImageView
        android:src="@drawable/ic_launcher"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content" />

    <RadioButton
        android:layout_alignParentRight="true"
        android:focusable="false"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content" />
	</com.xxx.widget.CheckableRelativeLayout>

下边关于ListView和Adapter的代码就省略了，实在没啥可说的。定义了这样的一个子布局之后，将ListView设定为singleChoice模式，点击后选择的状态就能够在布局中的RadioButton体现出来了。

**唯一需要注意的一点是，将RadioButton设定为不可获取焦点的，如果不这样设定，整个这一条将无法点击，ListView就不能正常处理选择状态了，具体怎么样试一下就知道。**