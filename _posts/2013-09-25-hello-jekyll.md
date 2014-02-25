---
layout: post
title: "Hello, Jekyll!"
description: ""
category: "jekyll"
tags: [Jekyll]
---
{% include JB/setup %}


Hello, Jekyll!
====


###发布新文章

    rake post title="title"

###本地测试

    jekyll --auto --server
>--auto会自动对变化进行更新

###排错处理

刚使用Jekyll Github的时候以为只要本地测试通过出了想要的效果就好了，
可是没想到却还是收到了Github的邮件:

    Your page is having problems building: page build failed

很悲剧，难怪等了半天还没有更新上去，原来是提交有问题。

本地调试命令：

    jekyll build --safe
>不知道--safe什么作用，也没查，github告诉我的。

将错误修正掉，然后再次提交就可以了。错误通常是markdown中的特殊字符问题。
