---
layout: post
title: "监听DOM加载完毕"
date: 2013-05-31 22:06
comments: true
categories: js 前端
---

我们知道javascript是可以在DOM加载完之前就执行的，如果对没有加载的DOM元素进行操作就会导致错误。浏览器的渲染顺序大致如下：

+ 解析HTML
+ 下载外部脚本样式
+ 脚本解析执行
+ DOM完全构建完成
+ 图片等外部资源加载
+ 网页加载完毕

那如何判断DOM是否加载完毕呢？使用body的onload事件进行监听？当然可以，但是body的onload是在所有内容，包括图片、flash啊等等内容全部加载后才触发，导致用户需要等待这部分内容加载后才能看到页面的动态内容，才能同页面进行交互，对于追(ti)求(gao)极(bi)致(ge)的程序猿来说，显然是不能接受的了，其实，只要检查如下3点足矣

+ document
+ document.getElementByTagName和document.getElementById
+ document.body

检查这三点就足够了。接着我们就可以写一个监听DOM何时加载完毕的函数了。

```javascript
    function domReady(f){

        // 假如DOM已经加载，马上执行函数
        if(domReady.done)
            return f();
        // 如果已经添加了一个函数
        if(domReady.timer){
            // 把它加入待执行函数清单
            domReady.ready.push(f);
        }else{
            // 初始化待执行函数的数组
            domReady.ready = [f];

            // 尽可能块的检查DOM是否已可用
            domReady.timer = setInterval(isDOMReady, 13);
        }
    }

    // 检查DOM是否可用
    function isDOMReady(){

        // 如果我们标记了已可用
        if(domReady.done)
            return false;

        // 检查DOM是否可用
        if(document && document.getElementByTag && document.getElementById && document.body){

            // 如果可用，停止检查
            clearInterval(domReady.timer);
            domReady.timer = null;

            // 执行所有函数
            for(var i=0; i<domReady.ready.length; i++){
                domReady.ready[i]();
            }

            domReady.ready = null;
            domReady.done = true;
        }
    }
```

这种方法使用起来很简单，而且不堵塞浏览器的加载。jquery的`ready()`函数功能使用的就是这种思想，当然实现的更加优雅和完备
