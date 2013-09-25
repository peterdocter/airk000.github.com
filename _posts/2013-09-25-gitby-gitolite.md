---
layout: post
title: "Git仓储权限管理by gitolite"
description: ""
category: "Git"
tags: [git]
---
{% include JB/setup %}

使用gitolite对git仓储进行权限配置
====

gitolite在近期做了很多代码改动，升级到了v3版本，而我使用的是v3.5.2。在《Git权威指南》中所提及的是v2版本，有很多东西已经不适合当前的v3版本，比如安装和用户自有仓储的配置，一些公用的部分有一些从书中摘抄而来。

###1.ssh协议
SSH 协议用于为 Git 提供远程读写操作，是远程写操作的标准服务，在智能HTTP协议出现之前，甚至是写操作的唯一标准服务。
ssh可用于远程登录，服务端需要安装openssh-server，客户端需要安装openssh-client。
之所以介绍ssh协议是因为gitolite以及gitosis都是基于ssh公钥认证的。

    $ ssh-keygen
该命令会在用户目录下.ssh目录下生成两个文件

    id_rsa
>私钥文件。是基于 RSA 算法创建。该私钥文件要妥善保管，不要泄漏。


    id_rsa.pub
>公钥文件。和id_rsa文件是一对儿，该文件作为公钥文件，可以公开。


    $ ssh-copy-id -i ~/.ssh/id_rsa.pub user@server
>将本地公钥提供给远程服务器，以达到无需口令直接登录的效果。实际上是将id_rsa.pub添加到authorized_keys中，直接操作authorized_keys效果一样。


**远程登录方法：**

    ssh user@server

**使用主机别名方法登录：**

编辑~/.ssh/config

    host server
    user admin
    port 22
    identityfile ~/.ssh/jiangxing #指定登录时使用的本地公钥

本地可以生成不同别名的公钥，方法是：

    ssh-keygen -f ~/.ssh/<filename>

###2.创建git用户

    $ sudo adduser --system --shell /bin/bash --group git
>创建git用户

    $ sudo adduser git ssh
>有的系统，只允许特定的用户组（如 ssh 用户组）的用户才可以通过 SSH 协议登录，这就需要将新建的 git 用户添加到 ssh 用户组中。

    $ passwd git
>设置密码

###3.生成ssh key

切换到git用户:

    $ su git
    $ ssh-keygen
    $ ssh-copy-id git@127.0.0.1
>本机公钥认证

###4.下载gitolite

    git clone git://github.com/sitaramc/gitolite

###5.安装配置/更新

我是安装在git用户根目录下的。
在根目录下创建bin文件夹
然后执行：

    ~/gitolite/install -to ~/bin
    mv ~/.ssh/authorized_keys ~/git.pub
    ~/bin/gitolite setup -pk ~/git.pub
成功后出现：

    初始化空的 Git 版本库于 /home/git/repositories/gitolite-admin.git/
    初始化空的 Git 版本库于 /home/git/repositories/testing.git/
>安装成功。

*所谓更新就是重新安装。*

###6.测试

还是在git用户下

    ssh git@127.0.0.1
如果返回类似这样的信息则代表gitolite工作正常：
    
    hello git, this is git@linux-dev running gitolite3 v3.5.2-4-g62fb317 on git 1.8.1.2
     
     R W    gitolite - admin
     R W    testing

###7.添加用户
在第5步可以看到，成功安装后gitolite会自动生成两个仓储:

- 一个是testing.git用来测试
- 另一个gitolite-admin就是用来管理gitolite的配置仓储

将gitolite-admin.git clone到本地，注意：还是在git用户下，因为当前只有git用户对其有读写权限。

    $ git clone git@127.0.0.1:gitolite-admin
成功clone到本地后，可以看到这个目录结构如下：

    git@linux-dev:~/gitolite-admin$ tree
    .
    ├── conf
    │   └── gitolite.conf
    └── keydir
         └── git.pub
    
    2 directories, 2 files
- conf是放置配置文件的目录
- gitolite.conf就是gitolite的配置文件，包含对用户、仓储、仓储权限的配置
- keydir目录用来放置所有的用户公钥
- git.pub为安装时setup -pk的那个用户公钥。

添加用户A，首先就是要把目标用户的公钥添加到keydir下，并重命名为该用户的用户名.pub。
目标用户A终端中执行:

    $ cat ~/.ssh/id_rsa.pub
    ssh - rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDVPPqRucnXGPOP2I6NbJ4wgg9vwb91mo9Q3AZJgbaK45Qz5UK71qM9JxL71jU3F2ogk1NHD0MCIlmmI50 / 1f1BHhd + aRjxEToTrxPXZM + xxxxxxxxxxxxxxxxxxxx+ zl7ftBPxtVYwSluJ + om4V4mbXT9 + uczRbCe1ejhYdg7vKYQV7K1VJ26hON8ztCRarL52Irq / 6a5It1Q78xv6Xf5F4mQOzUQsQp2EthtoA9XPiIybMjzNThDfbbKeW7kRZxBgi0RWLRYUSmc / UBNkQuub8l + II4S0FNhnUlNkmC / mUHKTqcjeS1fyJAkRcYC + fVTd4zqBNj1JupZfafpaeB userA@linux-dev

>将显示内容给git用户，或在git用户下粘贴来也可。目标用户名为userA，则最终应该以git用户身份将其保存在keydir目录下，命名为userA.pub。

然后添加用户相应权限。只要是在keydir下存在的用户，都属于@all用户组，其他用户组可通过在gitolite.conf自行定义。
如：

    @admin = git userA
    repo gitolite-admin
        RW+      =    @admin
     
    repo testing
        RW+      =    git
        RW       =    @all
admin用户组有两个用户git userA，分别对应keydir下的git.pub, userA.pub。

- gitolite-admin仓储的读/写/强制更新权限 只有admin用户组拥有；
- testing仓储的读/写/强制更新权限只有git用户拥有，其他所有在keydir下存在公钥的用户享有读/写权限。

###8.权限配置
权限配置在gitolite.conf中进行，注释用#表示。

C
>代表创建。仅在 通配符版本库 授权时可以使用。用于指定谁可以创建和通配符匹配的版本库。

R, RW, 和 RW+
>R 为只读。RW 为读写权限。RW+ 含义为除了具有读写外，还可以对 rewind 的提交强制 PUSH。

RWC, RW+C
>只有当授权指令中定义了正则引用（正则表达式定义的分支、里程碑等），才可以使用该授权指令。其中 C 的含义是允许创建和正则引用匹配的引用（分支或里程碑等）。

RWD, RW+D
>只有当授权指令中定义了正则引用（正则表达式定义的分支、里程碑等），才可以使用该授权指令。其中 D 的含义是允许删除和正则引用匹配的引用（分支或里程碑等）。

RWCD, RW+CD
>只有当授权指令中定义了正则引用（正则表达式定义的分支、里程碑等），才可以使用该授权指令。其中 C 的含义是允许创建和正则引用匹配的引用（分支或里程碑等），D 的含义是允许删除和正则引用匹配的引用（分支或里程碑等）。

\- （是的，这是一个减号，你没看错）
>是一条禁用指令。只对写操作起作用，即禁用用户的写操作。


接下来实际分析一个稍微复杂一些的配置文件

    1   @admin = git keven admin1 admin2
    2   @devteam = dev1 dev2 dev3 fish
    3 
    4   repo gitolite-admin
    5       RW+                 = git keven
    6 
    7   repo Projects/.+
    8       C                   = @admin
    9       RW                  = @all
    10 
    11  repo testing
    12      RW+                  =   @admin
    13      -                    =   fish
    14      RW      master       =   @dev
    15      RW+     dev          =   dev1
    16      RW      wip$         =   dev2

逐行解释：

    1: @admin用户组有git keven admin1 admin2四个用户
    2：@devteam用户组有dev1 dev2 dev3 fish四个用户
    4：对于gitolite-admin仓储
    5：git keven两个用户拥有读/写/强制更新的权限
    7：对于Projects下所有的git仓储（/.+代表递归所有）
    8：@admin用户组拥有创建仓储的权限
    9：所有人均可读/写
    11：对于testing.git
    12：@admin用户组拥有读/写/强制更新的权限
    13：fish是新手，对其屏蔽写的权限。因为其属@dev组，则还只剩下R 读的权限
    14：@dev用户组对master开头的分支拥有读/写权限
    15：dev1这个用户对dev开头的分支拥有读/写/强制更新的权限
    16：dev2这个用户对于wip分支（严格匹配）具有读/写权限


**冷门用法，需要用户对gitolite有一定了解**

有的时候用户可能需要在服务器端创建属于自己的仓储，这个时候就需要像下边这样：

    1  @admin = git userA admin1 admin2
    2  repo pub/CREATOR/.+$
    3      C       =   @all
    4      RW+     =   CREATOR
    5      R       =   @admin

每个用户都可以在users/<自己的用户名>目录下创建属于自己的仓储，而这个仓储，自己拥有完整的权限，管理员只有读权限。
>注：RW+ = CREATOR丢失会导致只能init空仓储而不能向上推送内容。

**用法：**
在用户shell中，进入要提交至服务器的仓储，执行：

    git push git@server:pub/<username>/somegit.git <branch>

用户可以通过ssh git@server perms对仓储权限进行设置，允许其他用户拥有写权限等。
>添加读权限是READERS，读写权限是WRITERS

操作：

    ssh git@server perms pub/<username>/somegit READERS user1
    ssh git@server perms pub/<username>/somegit WRITERS user2

###9.远程创建/删除仓储

#####创建

关于创建仓储，方法有三种：

**a. 登录远程服务器创建**

ssh登录服务器，切换至git用户，进入相关目录，创建某仓储：

    mkdir somegit.git
    cd somegit.git
    git init --bare
创建完毕

**b.修改gitolite.conf创建仓储**
打开gitolite-admin/conf/gitolite.conf，添加：

    repo testing2
        RW+    =  @all

保存修改，提交。

    git@linux-dev:~/gitolite-admin$ git commit -m 'add test2'
    [master b26be9a] add test2
    1 file changed, 4 insertions(+)
    git@linux-dev:~/gitolite-admin$ git push origin master
    Counting objects: 7, done.
    Delta compression using up to 2 threads.
    Compressing objects: 100% (3/3), done.
    Writing objects: 100% (4/4), 350 bytes, done.
    Total 4 (delta 1), reused 0 (delta 0)
    remote: 初始化空的 Git 版本库于 /home/git/repositories/testing2.git/
    To git@127.0.0.1:gitolite-admin
       0c409e4..b26be9a  master -> master
可以看到，gitolite会自动检测配置文件，发现目前没有的仓储会自动才创建。

**c.高端大气上档次**
对于通配符版本库，即repo Projects/.+类型的，在有创建权限的用户shell中，本地执行：

    mkdir somegit
    cd somegit
    git init
    git commit --allow-empty
    git remote add origin git@server:Projects/somegit.git
    git push origin master
gitolite会直接创建新的仓储。

#####删除：

1. 在conf/gitolite.conf中删除相关仓储配置信息（gitolite不会自动删除服务器上的文件，这点与add不同）；
2. 登录服务器删除需要删除的仓储。

#####重命名
同删除操作的步骤。

###10.mirror操作
http://gitolite.com/gitolite/mirroring.html

