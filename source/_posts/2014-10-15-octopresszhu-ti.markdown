---
layout: post
title: "Octopress主题"
date: 2013-12-20 17:52
comments: true
categories: 鼓捣
---

在Octopress默认主题的基础上进行了一些修改，把nav移到了顶端，同时在左侧增加了展示用户信息的sidebar。当浏览器宽度小于1152时，添加了响应式的布局。

源码地址：[github](https://github.com/wibud/Octopress-Theme-WinterBud)

如果对主题感兴趣，安装方式如下：

    $ cd octopress
    $ git clone git://github.com/wibud/Octopress-Theme-WinterBud.git .themes/WinterBud
    $ rake install['WinterBud']
    $ rake generate

![](/images/private/blog-screen.png)
