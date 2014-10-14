---
layout: post
title: "抛开兼容判断，可靠的DOM事件绑定"
date: 2013-04-03 17:17
comments: true
categories: js 前端
---

最近看到了一个很有意思的添加和删除DOM事件的方法，这个方法十分可靠，只是使用绑定事件的传统方法：`element.onclick=function(){...}`，但是我们知道，传统的绑定方法有一个很大的弊端，那就是一个元素的一个事件只能绑定一个处理程序，而新的方法很好的解决了这个问题。

让我们先回忆一下一般的我们如何跨浏览器的添加事件？

我想大部分应该都是使用如下的方法：

```javascript
    var eventUtil = {
        addEvent : function(type, ele, handler){

            // 判断是否支持标准(W3C)事件绑定
            if(ele.addEventListener){
                ele.addEventListener(type, handler, false);
            }
            // 判断是否是IE事件绑定
            else if(ele.attachEvent){
                ele.attachEvent("on"+type, handler);
            }
            // 传统事件绑定
            else{
                ele["on"+type] = handler;
            }
        }
    }
```

通过判断是否支持标准事件绑定还是IE事件绑定来决定使用哪种绑定事件的方式。

而新的这个很有意思的事件绑定方式就不同了，下面来看看

<!-- more -->

```javascript
    function addEvent(element, type, handler){

        // 为每个事件处理函数赋予一个独立的ID
        if(!handler.$$guid)
            handler.$$guid = addEvent.guid++;

        // 为元素建立一个事件类型的散列表
        if(!element.events)
            element.events = {};

        // 为每对元素/事件建立一个事件处理函数的散列表
        var handlers = element.events[type];
        if(!handlers){
            handlers = element.events[type] = {};

            // 存储已有的事件处理函数(如果已经存在)
            if(element["on"+type]){
                handlers[0] = element["on"+type];
            }
        }

        // 在散列表中存储该事件处理函数
        handlers[handler.$$guid] = handler;

        // 赋予一个全局事件处理函数来处理所有工作，使用传统绑定方法
        element["on"+type] = handleEvent;
    };

    // 创建独立ID的计数器
    addEvent.guid = 1;

    function removeEvent(element, type, handler){
        // 从列表中删除事件处理函数
        if(element.events && element.events[type]){
            delete element.events[type][handler.$$guid];
        }
    };

    function handlerEvent(event){
        var returnValue = true;

        // 获取事件对象
        event = event || fixEvent(window.event);    // fixEvent() for IE

        var handlers = this.events[type];

        // 依次执行每个事件处理函数
        for(var i in handlers){
            this.$$handlerEvent = handlers[i];
            if(this.$$handlerEvent(event) === false){
                returnValue = false;
            }
        }

        return returnValue;
    }

    // 增加一些IE事件对象缺乏的方法
    function fixEvent(event){
        // 增加W3C标准事件方法
        event.preventDefult = fixEvent.preventDefault;
        event.stopPropagation = fixEvent.stopPropagation;
        return evnet;
    };

    fixEvent.preventDefault = function(){
        this.returnValue = false;
    };

    fixEvent.stopPropagation = function(){
        this.cancelBubble = true;
    };'
```

不难看出使用传统的绑定方法可以在所有浏览器中工作，并且可以绑定多个事件处理函数。同时this关键字可以在所有绑定函数中使用，this指向的是当前绑定元素。瞬间逼格上涨，不过对比之前区分IE和标准的绑定方式，代码量的暴增可能也会让追求精简的童鞋抓狂吧。
