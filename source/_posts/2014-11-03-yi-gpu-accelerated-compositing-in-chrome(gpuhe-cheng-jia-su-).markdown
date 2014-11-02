---
layout: post
title: "[译]GPU Accelerated Compositing in Chrome（GPU合成加速）"
date: 2014-11-03 00:10:52 +0800
comments: true
categories: 前端
---

> 原文地址：http://www.chromium.org/developers/design-documents/gpu-accelerated-compositing-in-chrome

> 作者：Tom Wiltzius, Vangelis Kokkevis & the Chrome Graphics team

### 摘要

本文讲述了chrome中硬件加速合成（compositing[1]）的实现

### 简介：为什么使用硬件进行合成（compositing）

传统方式下，浏览器依赖CPU来渲染页面内容。随着GPU硬件能力的不断发展（包括一些超小型设备的硬件能力也有很大提升），人们开始试图使用GPU硬件能力来获得更好的性能和更少的电量消耗。使用GPU来渲染合成页面内容可以获得明显的速度的提升。

硬件合成有以下几个优点：

+ GPU在合成页面层（layer）上的效率要比CPU高很多，尤其是涉及大量像素的drawing[2]和compositing操作。当然，GPU设计出来本来就是做这些工作的。

+ Expensive readbacks aren't necessary for content already on the GPU (such as accelerated video, Canvas2D, or WebGL).

+ Parallelism between the CPU and GPU, which can operate at the same time to create an efficient graphics pipeline.

## Part 1：Blink Rending 基础

Blink Rending 引擎（由Google和Opera Software开发的浏览器排版引擎）的源码十分繁多和复杂。为更好理解在chrome中GPU加速是如何工作的，我们首先需要了解Blink引擎是如何渲染页面的。

### Nodes和DOM树

在Blink引擎中，页面内容是存储为由Node对象组成的树状结构，成为DOM树。每一个HTML element元素都有一个Node对象与之对应，DOM树的根节点永远都是Document Node。

### 从Nodes到RenderObjects

DOM树种每个node节点都有一个对应的RenderObject。RenderObject存储在与DOM树相对应的树形结构中——Render数（Render Tree）。RenderObject知道如何在屏幕上paint[3] Node内容，当然为实现这一操作，RenderObject会调用GraphicsContext来执行必要的draw[2]操作。其中，GraphicsContext就是负责将像素写入位图（bitmap）[4]中，这些位图最终会展示在屏幕上。

在老的实现方式下，大部分GraphicsContext调用会去调用一个SkCanvas或者SkPlatformCanvas等等，将其paint进一个软件位图（software bitmap）中（参考[这篇文章](http://www.chromium.org/developers/design-documents/graphics-and-skia)了解更多细节）。但是，现在painting不在放在主线程（main thread）（本文之后会讲到），这些命令现在会记录到一个SkPicture中。SkPicture是一个序列化的数据结构，它可以捕获命令并在之后重发这些命令，类似一个[display list](http://en.wikipedia.org/wiki/Display_list)

### 从RenderObjects到RenderLayers

每一个RenderObject都直接或间接（通过其父对象）的同一个RenderLayer相关联。

一般来说，拥有相同的坐标空间（比如：受相同CSS transform影响的）的RenderObjects，属于同一个RenderLayer。RenderLayer的作用就是保证页面元素以正确的顺序合成（composited），这样才能正确的展示元素的重叠以及半透明元素等等。会有一些情形，为一些特殊的RenderObjects创建一个新的RenderLayer。一下是常见的一定会新建RenderLayer的RenderObject：

+ 页面的根节点的RenderObject
+ 有明确的CSS定位属性（relative，absolute或者transform）
+ 是透明的
+ 有CSS overflow、CSS alpha遮罩（alpha mash）或者CSS reflection
+ 有CSS 滤镜（fliter）
+ 3D环境或者2D加速环境的canvas元素对应的RenderObject
+ video元素对应的RenderObject

值得注意的是，RenderObject同RenderLayer并不是一对一的。对于上述特殊的RenderObjects，它对应于为它所创建的心得RenderLayer，而其他的RenderObjects，对应于其第一个拥有RenderLayer的父RenderObject

RenderLayer也是一个树状的体系。根节点就是页面根元素所对应的RenderLayer，视觉上，每一个layer节点的后代都包含在父layer中。每一个RenderLayer的子layer保存在两个升序排列的列表中，分别是negZOrderList（负z-index的layers，即当前layer之下的layer）和posZOrderList（正z-index的layers，即当前layer之上的layer）

<!-- more -->

### 从RenderLayers到GraphicsLayers

为利用合成器（compositor），一些（不是全部）RenderLayers有独立的背景（backing surface）（有独立背景的层layer被认为是合成层（compositing layers））。如果RenderLayer是一个合成层，那么它又属于它自己的单独的GraphicsLayer，否则它和它的第一个拥有GraphicsLayer的父layer共用一个GraphicsLayer。如同RenderObject同Renderlayer的关系一样。

每一个GraphicsLayer都有一个GraphicsContext，其对应的RenderLayer会paint进GraphicsContext中。合成器（compositor）最终会负责，将由GraphicsContext输出的位图（bitmap[4]）合并成最终屏幕显示的图片。

虽然，在理论上，每一个独立的RenderLayer都可以paint进一个独立的背景中（拥有自己独立的背景，成为独立的合成层），但是，实际上，这样做十分消耗显存。在当前的Blink引擎的实现中，只有在如下的场景中，RenderLayer会是独立的合成层：

+ 有3D或者透视变换的CSS属性的层
+ 使用加速视频解码的video元素的层
+ 3D或者加速2D环境下的canvas元素的层
+ 插件，比如flash（Layer is used for a composited plugin）
+ 对opacity和transform应用了CSS动画的层
+ 使用了加速CSS滤镜（filters）的层
+ 有合成层后代的层
+ 同合成层重叠，且在该合成层上面（z-index）渲染的层

### 层压缩（Layer Squashing）

所有的规则都会有漏洞。正如上面提到的，GraphicsLayers会消耗内存和其他资源（比如一些饱受争议的操作，随着GraphicsLayer树的大小增长，会使CPU执行时间越来越长）。当一些RenderLayer同一个有着独立背景的RenderLayer重叠时，就会产生大量的额外的层，十分消耗资源。

我们把产生合成层的自身原因（比如有3D变换的层）称之为直接原因（direct compositing reasons）。为了防止上述所说的“层爆炸”，当很多元素在因直接原因产生的层之上时，Blink引擎，会将这些RenderLayers覆盖在“direct compositing reason”的RenderLayer上，同时将他们压缩（squash）成单一的一个备份。这就防止了由覆盖引起的层爆炸。更多细节请看[这里](https://docs.google.com/presentation/d/1WOhbWLkhMyo4vZUaHq-FO-mt0B2sejXw-lMwohD5iUo/edit#slide=id.g2a8a2080a_088)和[这里](https://docs.google.com/a/chromium.org/presentation/d/1dDE5u76ZBIKmsqkWi2apx3BqV8HOcNf4xxBdyNywZR8/edit#slide=id.p)

### 从GraphicsLayers到WebLayers到CC Layers

chrome在 src/webkit/renderer/compositor_bindings中实现了Web*Layer的接口

### The compositing Forest

总的来说，为rendering服务的有如下四种树形结构：

+ DOM Tree，基本的模型
+ RenderObject Tree，同DOM树的可见节点是一一对应的。RenderObject知道如何去paint其相对应的DOM节点
+ RenderLayer Tree，由RenderLayers组成，这些RenderLayer对应于RenderObject树的RenderObject。这种对应关系是一对多的。
+ GraphicsLayer Tree，由GraphicsLayers组成，这些GraphicsLayer对应于RenderLayer树的RenderLayer。这种对应关系是一对多的。

如下图所示：

![](/images/private/The-Compositing-Forest.png)

之后文章中所说的层（layer）都代指cc layer（chrome compositor layer）

## Part 2：The Compositor（合成器）

未完待续

## 附录C 术语表

[1] compositing：将RenderLayer的纹理（texture）合成为最终屏幕上的图片

[2] drawing：rendering术语，将像素点绘制到屏幕上

[3] painting：rendering术语，RenderObjects调用GraphicsContext API生成相应的视觉展现。生成元素呈现的像素，例如，一个有着灰色背景，有文字的元素，当浏览器paint它时，是决定哪些像素填充背景，哪些像素填充文字，然后浏览器将这些像素存入位图（bitmap）中。

[4] bitmap：内存或显存中一组像素值

[5] texture：应用于GPU 3D模型上的位图（bitmap）
