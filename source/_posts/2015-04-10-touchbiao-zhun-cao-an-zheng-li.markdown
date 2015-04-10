---
layout: post
title: "touch标准草案整理"
date: 2015-04-10 21:23:17 +0800
comments: true
categories: 前端
---

# touch事件

现在对于touch事件还没有正式标准出来的，只有一个推荐草案，[看草案戳这里](http://www.w3.org/TR/touch-events/)。当然，没有正式标准不代表没有浏览器支持，目前很多浏览器是可以支持touch的，可以从caniuse上看一下具体的支持情况

![](http://gtms01.alicdn.com/tps/i1/TB1jPv4FFXXXXXnaXXXHXUOSFXX-938-375.jpg)

[支持情况戳这里](http://caniuse.com/#search=touch)。

详细了解touch事件，还是很有必要看一下草案的（全英文好烦。。。心里一万头草泥马呼啸而过）

首先看一下两个概念，便于之后的阅读：

+ touch point（触点）：说白了就是咱手指点触摸屏时，与屏幕接触的那个坐标点
+ active touch point（活跃触点）：当前在屏幕上，且可以被浏览器追踪（捕获）到的触点。详细说来，当从一个触点上触发了touchstart事件，触点就激活了，变身成为活跃触点了，当在触点上触发了touchend和touchcancel事件时，触点就被移除或者不在被浏览器追踪了，此时这个活跃触点就挂掉了。

## 定义接口对象

### Touch：

> 描述touch point（触点）的对象。Touch对象创建之后，属性不可修改。

    interface Touch {
      readonly    attribute long        identifier;
      readonly    attribute EventTarget target;
      readonly    attribute long        screenX;
      readonly    attribute long        screenY;
      readonly    attribute long        clientX;
      readonly    attribute long        clientY;
      readonly    attribute long        pageX;
      readonly    attribute long        pageY;
	};

> Attributes属性详解：

+ clientX：触点相对视口的水平坐标（像素），不包括任何滚动偏移量

+ clientY：触点相对视口的垂直坐标（像素），不包括任何滚动偏移量

+ identifier：touch point（触点）的唯一识别码。当一个触点被激活（active touch point）时，它会被分配一个标示符，用于区分其他激活的触点（多点触摸的情景）当触点保持活跃（啥叫保持活跃？手指头放到屏幕上不离开，所对应的触点就一直是活跃的，不管咱是手指点着不动，还是不断滑动。当然触发touchcancel的话就不在这个范畴内了），引用到它的所有事件必须为其分配相同的标识符，保证每个触点有唯一的标示符。

+ pageX：触点相对视口的水平坐标（像素），包括滚动偏移量

+ pageY：触点相对视口的垂直坐标（像素），包括滚动偏移量

+ screenX：触点相对屏幕的水平坐标（像素）

+ screenY：触点相对屏幕的垂直坐标（像素）

+ target：eventtarget，和其他事件的target一样

### TouchList

> 定义了Touch对象的列表，不可修改，支持索引

    interface TouchList {
      readonly    attribute unsigned long length;
      getter Touch? item (unsigned long index);
	};

> Attributes：

+ length：不解释，都知道的

> Metods：

+ item：返回索引值对应的Touch对象

<!-- more -->
	
### TouchEvent

> 这个接口定义了touchstart、touchend、touchmove、touchcancel的事件类型。对象是immutable的。创建初始化后，属性不可修改

    interface TouchEvent : UIEvent {
      readonly    attribute TouchList touches;
      readonly    attribute TouchList targetTouches;
      readonly    attribute TouchList changedTouches;
      readonly    attribute boolean   altKey;
      readonly    attribute boolean   metaKey;
      readonly    attribute boolean   ctrlKey;
      readonly    attribute boolean   shiftKey;
	};

> Attributes：

+ altKey：boolean。Alt（按着）
+ changedTouches：TouchList类型，存放所有触点Touch。对于touchstart事件来说，changedTouches维护的是刚被激活的触点的列表（active touch event）。对于touchmove事件，changedTouches维护的是从上一次事件（touchstart）开始被移动的触点的列表。对于touchend和touchcancel事件，维护的是刚被移除的触点的列表。
+ ctrlKey：boolean。ctrl键是否被激活（按着）
+ metaKey：boolean。meta键是否被激活（按着）
+ shiftKey：boolean。shift键是否被激活（按着）
+ targetTouches：TouchList类型。作用于当前事件目标元素（target）上的触点列表。
+ touches：当前触摸屏幕的所有触点。

## TouchEvent

![TouchEvent](http://gtms04.alicdn.com/tps/i4/TB1Fy2lFpXXXXazbpXXa8NO8VXX-1197-176.jpg)

直接上图吧，就不翻译了，相(jiu)信(bu)大(xiu)家(yin)也(yu)看(xia)得(xian)懂(le)

### touchstart

当咱触摸屏幕时，就产生该事件。该事件的target必须是Element元素。

如果事件回调里调用preventDefault方法阻止了默认行为，则会阻止所有同该事件相关联的触点引发的其他行为，比如鼠标事件或滚动事件。简单说，就是咱为一个元素绑定了touchstart事件，回调中preventDefault了，那么手指点着这个元素然后拖动，是没有滚动效果的，被preventDefault阻止了。

### touchend

当用户从屏幕移除一个触点时触发该事件，简单说就是手指抬起来离开屏幕了。这个事件的target是触点第一次放置在屏幕上所对应的Element元素，即和touchstart事件的target是相同的。

即使触点离开了这个target element，target也是不会改变的。所有移除的触点只存放在changedTouches中，touches和targetTouches中是不会有移除的触点的。

### touchmove

当用户沿着触摸屏移动触点时触发该事件。这个事件的target是触点第一次放置在屏幕上所对应的Element元素，即和touchstart事件的target是相同的。

即使触点离开了这个target element，target也是不会改变的。

浏览器触发touchmove事件的频率应该是可以定义的（只是应该。。。现在还没有浏览器支持设置这个频率），并且可能是依赖硬件和其他实现细节的。

如果在touchmove事件回调中调用preventDefault方法，则会阻止所有同该事件相关联的触点引发的其他行为，比如滚动。

### touchcancel

触点收其他因素影响，比如：同步事件或者触点从文档窗口进入了能处理用户交互的非文档窗口（比如UA的本地用户界面，插件），不懂，求科普。

所有移除的触点存放在changedTouches中，touches和targetTouches中是不会有移除的触点的。

## Exemple

### touches and targetTouches of a TouchEvent，官方demo

``` javascript
	<div id='touchable'>
      This element is touchable.
  	</div>
  	<script>
    	document.getElementById('touchable').addEventListener('touchstart', function(ev) {      if (ev.touches.item(0) == ev.targetTouches.item(0))
      	{
          /**
           * If the first touch on the surface is also targeting the
           * "touchable" element, the code below should execute.
           * Since targetTouches is a subset of touches which covers the
           * entire surface, TouchEvent.touches >= TouchEvents.targetTouches
           * is always true.
           */          
            document.write('Hello Touch Events!');
      	}      
      	if (ev.touches.length == ev.targetTouches.length)
      	{
          /**
           * If all of the active touch points are on the "touchable"
           * element, the length properties should be the same.
           */          
           document.write('All points are on target element')
      	}      
      	if (ev.touches.length > 1)
      	{
          /**
           * On a single touch input device, there can only be one point
           * of contact on the surface, so the following code can only
           * execute when the terminal supports multiple touches.
           */          
           document.write('Hello Multiple Touch!');
      	}  }, false);
  	</script>
```

### changedTouches of a TouchEvent，官方demo

``` javascript
	<div id='touchable'>
      This element is touchable.
  	</div>
    <script>
    	document.getElementById('touchable').addEventListener('touchend', function(ev) {      
     	 /**
         * Example output when three touch points are on the surface,
         * two of them being on the "touchable" element and one point
         * in the "touchable" element is lifted from the surface:
         *
         * Touch points removed: 1
         * Touch points left on element: 1
         * Touch points left on document: 2
         */      
        	document.write('Removed: ' + ev.changedTouches.length);
        	document.write('Remaining on element: ' + ev.targetTouches.length);
        	document.write('Remaining on document: ' + ev.touches.length);  
    	}, false);
  	</script>
```

### target

``` javascript
	<div id='touchable'>
      This element is touchable.
  	</div>
  	<script>
    	document.getElementById('touchable').addEventListener('touchstart', function(ev) {      
     
	        console.log('touchstart target: ' + ev.target.id);
	        console.log('touchstart changedTouches: ' + ev.changedTouches[0].id);
	        console.log('touchstart touches: ' + ev.touches[0].id);
	        console.log('touchstart targetTouches: ' + ev.targetTouches[0].id);
	    }, false);
	
	    document.getElementById('touchable').addEventListener('touchend', function(ev) {      
	        
	        /*
	         * 手指滑出touchable元素的范围，target依然同touchstart的target相同
	         * 单触时，touches和targetTouches的length为0
	         */ 
	        console.log('touchend target: ' + ev.target.id);
	        console.log('touchend changedTouches: ' + ev.changedTouches[0].id);
	        console.log('touchend touches: ' + ev.touches.length);
	        console.log('touchend targetTouches: ' + ev.targetTouches.length);
	    }, false);
	
	    document.getElementById('touchable').addEventListener('touchmove', function(ev) {      
	        
	         /*
	         * target同touchstart的target相同
	         * 单触时，touches和targetTouches的length为0
	         */ 
	        console.log('touchmove target: ' + ev.target.id);
	        console.log('touchmove changedTouches: ' + ev.changedTouches[0].id);
	        console.log('touchmove touches: ' + ev.touches.length);
	        console.log('touchmove targetTouches: ' + ev.targetTouches.length);
	    }, false);
	  </script>
```

## 延伸阅读

[W3C Touch Events](http://www.w3.org/TR/touch-events/)

[W3C Pointer Events](http://www.w3.org/TR/pointerevents/)
