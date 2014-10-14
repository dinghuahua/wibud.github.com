---
layout: post
title: "float小细节"
date: 2013-07-08 10:28
comments: true
categories: css 前端
---

之前看了一篇文章，它设置basse.css的时候定义了很多的通用原子类，其中的.fl和.fr类是这样写的：`.fl{float:left;display:inline}`,`.fr{float:right;display:inline}`

### 为什么要在设置了`float:left`和`float:right`之外，还要设置`display:inline`呢？

其实这是为了解决IE6的双外边距的bug。在IE6中，如果对元素设置了浮动，同时又设置了margin-left或者margin-right的话，margin的值会加倍。解决这个问题的办法就是这是`display:inline`。所以在设计通用原子类的时候将`display:inline`引入.fl和.fr类中。

### 这样不会影响块级元素吗？

如果我们对块级元素赋予.fl或.fr类，其中的`display:inline`不会将块级元素变成行内元素了吗？其实这一点不用担心，`float:left`，`float:right`还有`position:absolute`，会让元素以`display:inline-block`的方式显示，就算我们设置了`display:inline`或者`display:block`，也仍然无效。

### display:inline-block

disploy的值除了inline和block之外，还有一个十分有用的值：inline-block（IE6,IE7不支持，但是可以实现相同效果）。

`display:inline-block`，从写法上就不难看出，它可肯定是行内元素特性inline和块级元素特性block的结合体，它具有块级元素的特点，可以设置长宽，margin和padding；又具有行内元素的特点，它不独占一行。

那在IE6和IE7下如何实现呢，我们知道IE为盒模型设计了一个专门的属性hasLayout，如果触发行内元素的hasLayout，就能让行内元素拥有块级元素的特性，达到`display:inline-block`的效果。幸运的是设置`display:inline-block`，就可以触发hasLayout。当然在IE6和IE7下还是有局限性，我们只能对行内元素实现`display:inline-block`的效果，对于块级元素就不行了。
