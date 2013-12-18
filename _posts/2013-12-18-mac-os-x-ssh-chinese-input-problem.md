--- 
layout: post
title: Mac OS X ssh连接Ubuntu汉字不能输入问题
---

今天遇到了一个十分头痛的问题，以至于一个下午都在解决这个问题。很简单，就是我用笔记本(OS X系统)在宿舍远程实验室的Ubuntu机器，其他一切OK，就是中文输入不能，每次输入中文的时候不是听到机器在beep就是出现一些怪异的东西。

在网上搜索了好久，首先确定这个是一个locale的问题，然后大多数网页给出的解决方案都是在Ubuntu中export以下几个环境变量

    export LANG="zh_CN.UTF-8"
    export LC_ALL="zh_CN.UTF-8"
    
然后我试了，完全没有用，还是中文输入不能 (以下省略2个小时坑网页瞎折腾的时间）。后来偶然想到既然ssh的时候中文显示的正常的，那么ubuntu的locale设置应该没有问题，这个时候才突然发现，这个不会是Mac的ssh的问题吧。Mac里重开一个终端输入locale果然发现

    LANG=
    LC_COLLATE="C"
    LC_CTYPE="UTF-8"
    LC_MESSAGES="C"
    LC_MONETARY="C"
    LC_NUMERIC="C"
    LC_TIME="C"
    LC_ALL=

果然是我被Mac坑了，果断在终端里面输入

    export LANG="zh_CN.UTF-8"
    export LC_ALL="zh_CN.UTF-8"

之后ssh上ubuntu，果然中文输入没有问题了。
