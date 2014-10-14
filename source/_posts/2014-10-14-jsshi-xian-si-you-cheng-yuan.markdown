---
layout: post
title: "JS实现私有成员"
date: 2013-03-30 12:19
comments: true
categories: js 前端
---

我们知道JavaScript是面向对象的语言，但是它同其他比如Java不同，它没有私有成员的概念，所有的对象属性都是共有的。不过我们可以通过闭包来实现私有变量。

### 私有变量

定义构造函数：

```javascript
    function Myobject(){
        // 私有变量
        var privateValue = "I'm private value";

        // 私有函数
        function privateFun(){
            return "I'm private function";
        }

        // 公有方法，可以访问私有成员的特权函数
        this.publicFun = function(){
            return privateValue + privateFun();
        }
    }
```

这样`object中的私有成员就对外不可见了，只能通过公共方法进行访问。

### 静态私有变量

之前的私有变量，每个使用构造函数创建的对象都有自己的私有成员，而静态私有变量是要所有的对象共享同一个私有变量，如下：

```javascript
    (function(){

        // 静态私有变量
        var staticPrivateValue = 1;

        // 静态私有函数
        function staticPrivateFun(){
            return "I'm static private function";
        }

        // 构造函数
        MyObject = function(){};

        // 公有方法
        MyObject.prototype.publicFun = function(){
            staticPrivateValue++;
            return staticPrivateFun();
        }

    })();
```

通过块级作用域来实现静态私有成员。其中`MyObjecct`构造函数定义时，没有声明就赋值初始化，导致`MyObjecct`变成了全局对象，这样在块级作用域外就可以访问到`MyObjecct`构造函数，而块级作用域中的静态私有变量只有定义的公有方法可以访问，同时对`MyObjecct`所有对象都是共享的。

<!-- more -->

### 单例

单例即是只能创建一个实例对象，不需要构造函数，直接返回一个对象即可实现单例。

```javascript
    var singleObj = function(){

        // 私有成员
        var privateValue = 1;
        function privateFun(){
            return "Hello";
        }

        // 返回对象
        return {
            // 公用方法
            publicFun : function(){
                privateValue++;
                return privateFun();
            }
        }
    }();
```

通过使用一个返回对象的匿名函数来实现单例模式和私有成员。
