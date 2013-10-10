---
layout: post
title: "Git生词本"
description: ""
category: "git"
tags: [git]
---
{% include JB/setup %}

Git生词本
====

    git rev-list [since]..[to]
since为旧的，to为新的。类似git log，但是只显示SHA值，如：

    $ git rev-list fb99c71..730ce4c
    730ce4c3c2128b489176513253c1f30b378c694c
    745a39ba3dca7699ac057887e79deeb47e2bf8a5
    efc986c5081fc9a504b698672d955300a99f24bb
    5e0ee145759b4a197bea347ad77e1ad1609efd32
    70df18944a667329ec894a1df1e77381c5b0c09a
    0836a22d38f4fa29d3cbc543fcd7a42813ec052d
    b6a16e6390b12b019a7e5e297639e41310375c93
    351fe2c793437e1d8a0b222f8478c74cf60ae034

不知道since to的情况下也可以这样使用：

    git rev-list [branch] [-num]

如：

    git rev-list master -1
>这样就是显示master分支的最后一个提交的SHA值

查看某个提交的提交信息：

    git caf-file commit c8ab16 | sed '1,/^$/d'

###核弹级选项：filter-branch

#####需求1：

需要删除一个在很多commit中都已经存在的文件，这个文件可能是一个密码文件、也可能是一个比较大的没有的二进制文件，怎样在不重做git仓储的情况下做出这样的修改？

    git filter-branch --tree-filter 'rm -f passwords.txt' HEAD
>能将HEAD所指向的分支中每个commit的点都执行一个删除passwords.txt的操作（无论存在与否），然后自动重写commit。

比如常用的可能就会有：

    git filter-branch --tree-filter 'rm -f *~' HEAD

如果要在所有分支上都运行一遍的话就加一个--all。

#####需求2：

将子目录src设置为git新的根目录。

    git filter-branch -subdirectory-filter src HEAD

#####需求3：

也许你的项目一开始忘了修改git config里的user.name user.email，但是你要发布了（或者要跟其他开发者协作），你希望能能够在不重做git仓储的情况下改变之前那些默认用户邮箱的author信息。

    git filter-branch --commit-filter '
    if [ "$GIT_AUTHOR_EMAIL" = "user@localhost" ];
    then
        GIT_AUTHOR_NAME="Keven Liu";
        GIT_AUTHOR_EMAIL="airk908@gmail.com"
        git commit-tree "$@"
    else
        git commit-tree "$@"
    fi' HEAD
