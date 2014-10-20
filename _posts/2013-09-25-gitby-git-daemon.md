---
layout: post
title: "Git服务器搭建by git-daemon"
description: ""
category: "git"
tags: [git]
---

使用git-daemon进行git服务器搭建
===

###1.安装git-daemon
前提是已经安装git

    sudo apt-get install git git-core

然后安装git-daemon

    sudo apt-get install git-daemon-run

###2.配置

安装完成，修改配置文件 /etc/service/git-daemon/run

    #!/bin/sh
    exec 2>&1
    echo 'git-daemon starting.'
    exec chpst -ugitdaemon \
      "$(git --exec-path)"/git-daemon --verbose \
        --export-all --base-path=/home/username/git

修改--base-path字段为自己放置git项目的目录。

###3.重启服务

**方法一：**

    ps -eaf | grep -v grep | grep git

kill git-daemon进程
>runsv会自动对git-daemon进行重启

**方法二：**


    sudo sv down git-daemon
    sudo sv up git-daemon

###4.使用
在/home/username/git目录下初始化一个testing仓储。

    git clone git://127.0.0.1/testing
服务重启成功后用git://访问，会自动采用git-daemon进行引导。

git-daemon搭建完毕，提供git仓储只读服务。
-----
