---
layout: post
title: "带圆角的图片"
description: "使图片带有圆角。"
category: "Android开发便签"
tags: [Android]
---

###要完成的效果

<svg width="100%" height="100%" version="1.1"
xmlns="http://www.w3.org/2000/svg">
<rect x="100" y="20" width="200" height="130"
rx="20" ry="20"
style="fill:blue;stroke:black;stroke-width:1;fill-opacity=0.6;
stroke-opacity=0.3" />
</svg>

###实现代码


	BitmapShader shader = new BitmapShader(bitmap, TileMode.CLAMP, TileMode.CLAMP);

	Paint paint = new Paint();
	paint.setAntiAlias(true);
	paint.setShader(shader);

	RectF rectF = new RectF(0f, 0f, getWidth(), getHeight());
	
	canvas.drawRoundRect(rectF, 10f, 10f, paint);


- 定义BitmapShader，内容填充模式为固定，还有重复`REPEAT`、镜像`MIRROR`
- 将图形设置到Paint画笔中
- 定义绘制的矩形区域
- canvas绘制圆角矩形，使用画笔绘制给定的矩形区域，并且带有10像素x轴圆角和10像素y轴圆角