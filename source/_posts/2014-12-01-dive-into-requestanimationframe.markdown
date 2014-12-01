---
layout: post
title: "dive into requestAnimationFrame"
date: 2014-12-01 00:14:36 +0800
comments: true
categories: 前端
---

## 介绍

requestAnimationFrame(以下简称raf)方法，字面意思上看是：请求动画帧；较官方的解释是：通知在系统准备好绘制动画帧时调用该帧，从而为创建动画网页提供了一种更平滑更高效的方法；那通俗一点就是：告诉浏览器说 “我这里有一帧（一些操作），需要你帮我绘制（执行）一下”，“ok，没问题，我挑个合适的时机给你绘制这一帧（执行这些操作）”

![](/images/private/raf-1.jpg)

+ 一、浏览器的绘制（执行）时机是嘛时候
	
	首先，我们需要了解的是，页面的绘制由CPU或者GPU进行的，但是其绘制频率受限于我们的显示器的刷新频率，一般显示器的刷新频率是60HZ，所以对应于绘制频率最高就是60fps（frame per second）。于是60fps也成了检验性能的一个重要的指标，be closer be better，通常来说，在30fps到60fps之间都是可以接受的。
	
	60fps意味着我们执行每一帧的时间只有大约16.7ms（1000/60），这也是常见的使用setTimeout和setInterval实现动画时所设置的间隔时间。那对于raf来说呢，我们将绘制（执行）时机交给了浏览器自己进行判断，浏览器在合适的时机来绘制（执行）我们提供给浏览器的帧（操作）。浏览器如何决定这个合适时机的呢？（均基于chrome环境）
			
	> raf最大的调用频率不会超过60fps
		
	主观上来说，60fps已经达到了很好效果，没必要用到更快的更新频率了。貌似这一点也是chromium对于raf的设计原则之一，为了印证这一结论，特地去扒了扒chromium中raf的执行流程
		
	![](/images/private/chromium-request-anim-frame.png)
		
	从流程图可以看到
		
		if (animation update pending and >= 16ms since last update)
			post task to anticipate next frame
		else
			post task to retry in (next_frame - now)ms
				
	如果有等待的animation更新且距离上一次更新超过16ms，则会post task，来准备下一帧，也就是说
		
	> 针对chrome而言，无论屏幕的刷新频率是多少，raf回调函数的执行频率不会超过60次/秒
		
	找了大量的资料也没有找到raf回调的执行时机是如何计算的，唯一的途径只有看源码了，不过源码是C++，出了大学课堂就再没用过，全忘干净了。。。，大体看了看，貌似原理是：如果上一次raf的回调执行时间过长，那么触发下一次raf回调的时间就会缩短，反之亦然，目的是保证动画不丢帧。（如有错误，请轻拍~~）
		
	其实，raf不需要关心屏幕什么时候刷新，也并不是在屏幕刷新的时候进行绘制，而是保证在屏幕刷新间隔内不会有两个帧的绘制而导致前一个帧丢失。
	
<!-- more -->
		
+ 二、raf如何使动画更平滑更高效

	为何说raf能使动画更平滑更高效呢？
	
	> raf并非升帧，并非加快执行速度，而是适当时候降帧，防止并解决丢帧问题
	
	如此，虽然降帧了，比如降到30fps，但是因为视觉暂留的原理，我们依然会觉得动画是连贯的，而不会因为丢帧二导致动画断续、卡顿显示。从而使动画更加平滑。
	
	之所以说raf使动画更平滑和高效，是相对于原来使用setTimeout或setInterval的实现来说的，下节会详细讨论。

在这里不谈requestAnimationFrame(以下简称rAF)用法，具体请参考[MDN:Window.requestAnimationFrame()](https://developer.mozilla.org/en-US/docs/Web/API/window.requestAnimationFrame)。

## setTimeout(setInterval) VS requestAnimationFrame

以下将setTimeout和setInterval统称为Timer。

![](/images/private/raf-2.png)
	
上面第一行表示大多数监视器上显示的 16.7ms 显示频率，而下面一行表示的 10ms的Timer。每个第三个图形都无法绘制（由红色箭头指示），因为在显示器刷新间隔之前发生了其他绘制请求，也就是说第三个图形还没绘制，第四个图形的绘制请求就来了，导致下一次绘制，只会绘制第四个图形。这种过度绘制的情况会导致动画断续显示，因为所有第三帧都会丢失。
	
那我们把Timer的时间间隔设置为16.7ms呢？

+ 首先，Timer时间间隔是依靠浏览器的时钟来计算的，时钟的精确度是取决于时钟的更新频率的，比如，在IE8-中，时钟的更新频率是15.6ms，也就是说如果你设置Timer的时间间隔是16.7ms，那也要等两个15.6ms才会执行Timer的回调。目前chrome和IE9+的更新频率是4ms，如果是笔记本，在使用电池的情况下，时钟的更频率会变的更低。

+ 其次，就算Timer时间间隔不依赖浏览器时钟更新或者说能和时钟更新保持一致，但是我们知道，Timer的实现机制是在设置的时间间隔后往任务队列里push执行回调函数的任务，如果在Timer的回调函数执行任务之前存在大量其他任务的话，任务队列是需要执行完这些任务，才会执行Timer的任务，就导致实际执行回调的时间要大于我们设置的时间间隔。

+ 最后，就算没有之上的两个问题，我们的Timer可以在时间间隔上准确执行，依然会有丢帧的问题，比如我们将Timer时间间隔设置为16.7ms，屏幕的刷新频率是60Hz（时间间隔16.666...7），Timer更新频率同屏幕刷新频率是有误差的，随着动画运行时间的推移，误差会不断累加，大约在500s后，会发生丢帧。

其他问题：

Timer可能会导致动画过度绘制，浪费 CPU 周期以及消耗额外的电能等问题。而且，即使看不到网站，特别是当网站使用背景选项卡中的页面或浏览器已最小化时，动画都会频繁出现。参见这篇[翻译文章](http://segmentfault.com/blog/humphry/1190000000386368)，标签页闲置时，时间间隔如下：

|                  | setInterval | requestAnimationFrame |
| ------------- |:-------------:| -----:|
| IE10+ | 无影响(时间间隔不变) | 暂停 |
| IE9- | 无影响 | 不支持 |
| Safari | >= 1s | 暂停 |
| Firefox | >= 1s | 1s - 3s |
| Chrome | >= 1s | 暂停 |
| Opera | >= 1s | 暂停 |

其实，在现代浏览器中，对于Timer也是做了不少优化的，单从fps角度上来说，Timer和raf相差不大（参考[这篇测试文章](http://segmentfault.com/blog/humphry/1190000000400068)），raf比之于Timer，最大的优势是解决了丢帧问题~~

对了，从setTimeout切换至raf非常容易，他们都是单一回调，以下是一个很完善的兼容的方法，可以放心大胆的用了，妈妈再也不用担心我的兼容问题了~~

	(function() {
		var lastTime = 0;
    		var vendors = ['ms', 'moz', 'webkit', 'o'];
    		for(var x = 0; x < vendors.length && !window.requestAnimationFrame; ++x) {
        		window.requestAnimationFrame = window[vendors[x] + 'RequestAnimationFrame'];
        		window.cancelAnimationFrame = window[vendors[x] + 'CancelAnimationFrame'] || window[vendors[x] + 'CancelRequestAnimationFrame'];
    		}

    		if (!window.requestAnimationFrame) {
        		window.requestAnimationFrame = function(callback, element) {
            		var currTime = new Date().getTime();
            		var timeToCall = Math.max(0, 16.7 - (currTime - lastTime));
	            	var id = window.setTimeout(function() {
      	          		callback(currTime + timeToCall);
           		 	}, timeToCall);
            		lastTime = currTime + timeToCall;
	            	return id;
        		};
    		}
    		if (!window.cancelAnimationFrame) {
	    		window.cancelAnimationFrame = function(id) {
            		clearTimeout(id);
            	};
    		}
	}());

## CSS Animation VS requestAnimationFrame

### 一、性能

一般我们会认为CSS动画的性能肯定会比js帧动画性能要好，因为CSS动画可以被硬件加速，通过GPU可以提升动画性能。真的是这样吗？

其实硬件（GPU）加速的应该是层（GraphicLayer）的合成，而CSS动画可以让应用动画的层成为一个独立的层（GraphicLayer），成为独立的层简单来说有如下好处：

+ 每个GraphicsLayer都有一个GraphicsContext，GraphicsContext会输出该层的位图，交由GPU合成，比CPU要快
+ 当需要repaint时，只需要repaint自己，不会影响到其他的GraphicsLayer。repaint完之后，只需要通过GPU同其他层合并下（composite layers）
+ 对于CSS transform等，不需要repaint

而元素（准确的说应该是RenderLayer）想要拥有独立的GraphicLayer，满足如下条件即可：

+ Layer has 3D or perspective transform CSS properties
+ Layer is used by video element using accelerated video decoding  
+ Layer is used by a canvas element with a 3D context or accelerated 2D context
+ Layer is used for a composited plugin
+ Layer uses a CSS animation for its opacity or uses an animated webkit transform
+ Layer uses accelerated CSS filters 
+ Layer has a descendant that is a compositing layer  
+ Layer has a sibling with a lower z-index which has a compositing layer (in other words the layer overlaps a composited layer and should be rendered on top of it)（chrome中做了优化，方式该情况下出现的层过多）

详细可戳这里[GPU Accelerated Compositing in Chrome](http://www.chromium.org/developers/design-documents/gpu-accelerated-compositing-in-chrome)

也就是说即使是用js帧动画，我们也可以通过让动画元素拥有独立的GraphicLayer，来使用GPU合成加速。这里需要注意的是，独立的GraphicLayer会占用内存（显存），尤其是在手机上，大部分手机的显存和内存是共享的，GraphicsLayer过多导致内存占用过多的话，会使手机变卡。

所以说在性能方便CSS动画和js帧动画直接差别不大，而且某些CSS动画可能也会给GPU带来过大的压力，导致动画性能并不高。Adobe的[这篇文章](http://blogs.adobe.com/webplatform/2014/03/18/css-animations-and-transitions-performance/)中，对两类动画做了分析，本别是transition: height和 transition: transform，其中用CSS动画改变元素的高度，动画性能并不高，有兴趣去看一下这篇文章，这里就不赘述了。（文章中，paint操作现在应该是在合成线程composite thread中进行的，而不是在主线程中）

就目前来说，对于CSS transform、CSS opacity、CSS filter，使用CSS动画会比js帧动画好，因为针对这些属性的CSS动画并不阻塞执行js的主线程，而使用js帧动画势必会占用主线程。

### 二、控制性

这个就不需多说了，相信大家都知道，肯定是js帧动画胜出了。

### 三、浏览器支持情况

raf的支持情况可以去[caniuse](http://caniuse.com/)去查下，raf支持情况没有CSS动画好，在android4.4+才支持raf，而在android4.4下，使用js帧动画只能用setTimeout或者setInterval了，效果估计就没有CSS动画那么给力了。

## 参考文档

+ [Rendering Architecture Diagrams](http://www.chromium.org/developers/design-documents/rendering-architecture-diagrams)
+ [MDN:Window.requestAnimationFrame()](https://developer.mozilla.org/en-US/docs/Web/API/window.requestAnimationFrame)
+ [CSS vs. JS Animation: Which is Faster?](http://davidwalsh.name/css-js-animation)
+ [CSS3 Transitions vs GSAP: Cage Match](http://greensock.com/transitions/)
+ [Leaner, Meaner, Faster Animations with requestAnimationFrame](http://www.html5rocks.com/en/tutorials/speed/animations/?redirect_from_locale=zh)
+ [CSS animations and 
transitions performance: 
looking inside the browser](http://blogs.adobe.com/webplatform/2014/03/18/css-animations-and-transitions-performance/)
