---
layout: post
title: "JavaScript作用域与闭包"
date: 2013-03-28 20:11
comments: true
categories: 前端 js
---

在读《JavaScript权威指南》和《JavaScript高级程序设计》的时候感觉俩本书对JS的作用域讲的都不太清晰，于是就自己总结了一下。

### 执行环境和作用域

执行环境就是JS代码执行时所在的环境，每个执行环境都有一个与之关联的__变量对象__，这个对象中就保存着执行环境中的__所有变量和函数__。全局执行环境是最外围的执行环境，也就是window对象了，所以我们定义的全局变量和函数都保存在window对象中。

对于函数来说，每个函数都有一个自己执行环境。

当代码在环境中执行时，会创建一个作用域链，其实就是代码所在的一个个执行环境（嵌套的）的变量对象（函数除外，之后讨论）做成的链，链的最前面当然是变量直接所在的执行环境的变量对象，之后依次是嵌套包含的上级执行环境的变量对象，全局执行环境的变量对象位于链的末尾。使用一个变量时就会从变量的作用域链头部开始往后查找直到找到这个变量。

而__函数__就有些特殊了，函数是通过词法来划分作用域的，在函数定义时，它的作用域链就保存了起来，而并不是执行函数的时候创建的作用域链，这也就是为什么会有闭包这个现象了。函数的作用域链中的变量对象是函数的调用对象（活动对象），调用对象中保存有函数的参数，函数中定义的局部变量。这里注意的是this是一个关键字，并不是调用对象的一个属性，this和arguments都是在函数__执行时__，调用对象取得的，this代表的就是函数据以执行的执行环境

### 闭包
闭包的__定义__：有权访问另一个函数作用域中的变量的函数。简单讲就是在一个函数内部又创建了一个函数。下面讲讲闭包需要注意的问题：

#### 1. 内存泄漏

```javascript
    function outer(param){

        return function(){
            return param;
        }
    }
    // 创建函数
    var inner = outer();

    // 调用函数
    var result = inner();
```

上面就是一个简单的闭包的例子，正常来看是没什么问题，`outer()`函数返回后它的作用域链会被销毁，但是它内部的匿名函数的作用域链依然引用`outer()`函数的调用对象，致使`outer()`函数的调用对象依然留在内存中。所以我们需要解除对匿名函数的引用。

    // 解除对匿名函数的引用，以便释放内存
    inner = null;

<!-- more -->

再看下面这个例子：

```javascript
    function handler(){
        var ele = document.getElementById("someone");
        ele.onclick = function(){
            alert(ele.id);
        }
    }
```

如果闭包的作用域链中保存着一个HTML元素，那么将使得该元素无法被销毁。上面的例子创建了一个元素的`onclick`事件的处理程序的闭包，由于匿名函数一直保持着`handler()`函数的调用对象，使得只要匿名函数存在，对于HTMl element元素的引用就至少为1,而它所占用的内存也就一直得不到回收。这就需要我们手动显示的收回引用。

```javascript
    function handler(){
        var ele = document.getElementById("someone");
        var id = ele.id
        ele.onclick = function(){
            alert(id);
        }

        ele = null;
    }
```

#### 2. this对象问题

> 匿名函数的执行环境具有全局性，因此this对象通常指向window

在闭包中使用this对象可能也会导致问题。看下面的例子：

```javascript
    var name = "window";
    var object = {
        name : "object";
        getName : function(){
            return function(){
                return this.name;
            }
        }
    }

    alert(object.getName()());      // "window"
```

会alert出"window"，是不是感觉很奇怪？

上面提到过，this对象是函数被调用时，调用对象取得的，内部函数在沿着作用域链搜素this的值的时候，首先就会搜索到它自己的调用对象，就可以找到this变量了，这样根本就不可能访问外部函数的this变量。解决这个问题很简单，只要将this对象保存到闭包访问的到的变量里就可以了：

```javascript
    var name = "window";
    var object = {
        name : "object";
        getName : function(){
            var that = this;
            return function(){
                return thit.name;
            }
        }
    }
```
