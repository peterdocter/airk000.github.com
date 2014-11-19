---
layout: post
title: "RelativeLayout代码中设置LEFT_TO/RIGHT_TO/BELOW/ABOVE等"
description: "RelativeLayout使用xml定义相对位置关系很简单，想要在代码中实现相同功能就需要注意一下了。"
category: "Android开发便签"
tags: [Android]
---

###自定义RelativeLayout

假设有个简单的小需求，让RelativeLayout添加View时像LinearLayout一样能够横向或竖向排列，并不要重叠，而且是通过addView后直接就是这种效果。

可以复写addView：

	@Override
    public void addView(View child) {
        View last = getChildAt(getChildCount() - 1);
        if (last != null) {
            ViewGroup.LayoutParams lp = child.getLayoutParams();
            if (lp == null || !(lp instanceof LayoutParams)) {
                lp = new LayoutParams(ViewGroup.LayoutParams.WRAP_CONTENT, ViewGroup.LayoutParams.WRAP_CONTENT);
            }
            ((LayoutParams) lp).addRule(mOrientation == VERTICAL ? BELOW : RIGHT_OF, last.getId());
            child.setLayoutParams(lp);
        }
        super.addView(child);
    }

这里RelativeLayout初始化时候自动添加进了一个Button作为其第一个child。而上边复写的代码经过实验证明并为生效，所有后添加进去的View都是重合在一起的。

###Why？

OK，代码上看，LayoutParams设置的也没有问题，child也并不需要我们进行处理，为什么输出的结果还是不一样呢？就是因为Android系统中的ViewGroup在通过代码addView时并不会给子View申请ID，而在xml中的所有id都会生成到public.xml中供应用使用。因为这样的习惯思维，导致可能认为addView后ViewGroup也会给子View申请ID，最终出现了错误。

###修复，成功！

既然已经发现问题是因为没有ID导致的，那就赋予子View们ID，而这个ID在其所在的ViewGroup界内还必须是唯一的。对addView的复写代码添加这样一行：

	child.setId(getChildCount());
	
这样就保证了child View既有ID又不会重复。测试看一下效果，成功了！

DONE～