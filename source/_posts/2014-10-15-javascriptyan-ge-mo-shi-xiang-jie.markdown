---
layout: post
title: "JavaScript严格模式详解"
date: 2014-01-09 15:17:48 +0800
comments: true
categories: js 前端
---

## 一、简介

严格模式（strict mode）是在ES5（ECMAScript 5）中引入的一种新的JavaScript运行模式，该模式下，禁止了很多JavaScript中问题较多、容易出错、影响应能的特性，目的是让JavaScript在更为严格的条件下运行，减少怪异行为和语言不合理的地方，提高编译运行性能（比如禁用with语句），增强了代码本身的安全性。

在ES5之前的版本是不支持严格模式的，但是严格模式有很好的向后兼容性，因为低版本标准的引擎会忽略严格模式的声明，这个会在之后进行介绍。

想了解各浏览器对于严格模式的支持情况戳这里[ECMAScript 5 compatibility table](http://kangax.github.io/es5-compat-table/)，其中还统计了ES5新增特性的兼容情况。

## 二、声明

声明调用严格模式十分简单，也有些怪异，使用字符串字面量：

``` javascript
    "use strict";
```

之所以使用字符串字面量好处就是可以向后兼容，对于不支持严格模式的引擎，执行这条语句没有任何作用和影响。当然，如果我们使用了严格模式，那么就需要使用支持严格模式的引擎进行测试，使用低版本的引擎无法检测出严格模式下的错误。

声明方式有如下两种：

+ 1、声明整个脚本

将`"use strict;"`在脚本文件第一行，且必须是第一行，否则严格模式将无效。

> 在`"use strict;"`前不能有任何执行语句，即使是一个空的分号，都会使严格模式失效。

`<script>`包含的脚本块和引入的外部文件对于严格模式来说都是相互独立的，也就是说，一个脚本块或脚本文件设置成严格模式执行，并不会影响到其他的脚本块和脚本文件。

``` javascript
    <script>
        "use strict";
        console.log("严格模式");
    </script>
    <script>
        console.log("正常模式");
    </script>
```

+ 2、声明单个函数体

单个函数的声明和整个脚本的类似，需要将`"use strict;"`放在函数体的第一行。这样该函数就会以严格模式运行。

``` javascript
    function f(){
        "use strict";
        // ...
    }
```
<!-- more -->

在大型多人合作的项目中使用严格模式，就需要多加注意了。因为，在开发中可能使用了多个脚本文件，而在部署时，会将多个文件合并成一个，如果有的脚本文件使用严格模式，而有的并没有使用，那么合并起来就可能导致意想不到的问题。比如，需要使用严格模式的地方失效了，而不需要使用严格模式的地方反而运行在严格模式下了。

对于此类问题，有如下两种的解决方法：

+ 1、不将严格模式运行的文件和正常模式运行的文件合并起来。

这应该是最简单的解决方法了，那么你在部署的时候就至少需要两个独立的脚本文件。但这样无疑会对你的文件管理增加负担。

+ 2、使用独立作用域（立即调用的函数表达式）

将脚本文件内容包含再立即调用的函数表达式中，就好像模块一样。

``` javascript
    (function(){
        "use strict";
        function f(){
            // ...
        };
        // ...
    })();
```

这样合并成一个文件以后就不会相互影响了

## 三、严格模式的限制

以下列出了严格模式最主要的限制。


+ 1、变量必须使用var声明

正常模式下，未声明的变量，默认是全局变量，而在严格模式下，必须对变量进行声明，否则会报错：“SCRIPT5042：严格模式下未定义变量”。

``` javascript
    "use strict";
    testValue = 1;  // 报错
    for(i=0; i<10; i++){    // 报错
        // ...
    }
```

+ 2、禁止写入只读属性

正常模式下，对对象的只读属性进行赋值，不会出现错误，只是会默认失效，而在严格模式下，写入只读属性（包括使用get进行读取的属性）会报错：“SCRIPT5045：严格模式下不允许分配到只读属性”。


``` javascript
    "use strict";
    var testObj = Object.defineProperties({}, {
        prop1: {
            value: 10,
            writable: false // by default
        },
        prop2: {
            get: function () {}
        }
    });
    testObj.prop1 = 20;     // 报错
    testObj.prop2 = 30;     // 报错
```

+ 3、禁止为不可扩展的对象新增属性

正常模式下，为不可扩展的对象添加属性不会报错，只会默认失效，而在严格模式下，会报错：“SCRIPT5046：无法为不可扩展的对象创建属性”。

``` javascript
    "use strict";
    var testObj = new Object();
    Object.preventExtensions(testObj);
    testObj.name = "Bob";   // 报错
```

+ 4、禁止删除变量、函数、参数和configurable为false的属性

在严格模式下，只有configurable特性设置为true的对象属性才可以被删除，否则将报错：“SCRIPT1045：严格模式下不允许对 <表达式> 调用 Delete”。

``` javascript
    "use strict";
    var testvar = 15;
    function testFunc() {};
    delete testvar;     // 报错
    delete testFunc;    // 报错

    Object.defineProperty(testObj, "testvar", {
        value: 10,
        configurable: false
    });
    delete testObj.testvar;     // 报错
```

+ 5、禁止对象属性名重复

正常模式下，如果对象的属性名重复，那么后一个属性的值会覆盖前一个同名属性的值，而在严格模式下，如果对象有同名属性，则会报错：“SCRIPT1046：严格模式下不允许一个属性有多个定义”。

``` javascript
    "use strict";
    var testObj = {
        prop1: 10,
        prop2: 15,
        prop1: 20
    };  // 报错
```

+ 6、禁止函数参数名重复

正常模式下，如果函数有多个重名参数，则可以使用argument[i]来读取相应的参数，而在严格模式下，如果函数存在参数重名，则会报错：“SCRIPT1038：严格模式下不允许正式参数名称重复”。

``` javascript
    "use strict";
    function testFunc(param1, param1) {
        return;
    };  // 报错
```

+ 7、保留关键字

为了向将来Javascript的新版本过渡，严格模式新增了一些保留字：implements, interface, let, package, private, protected, public, static, yield。

在严格模式下，使用这些这些保留字做变脸或函数名都会报错：“SCRIPT1050：无法使用标识符的未来保留字。 严格模式下将保留标识符名称。”

> 此外，ECMAscript第五版本身还规定了另一些保留字（class, enum, export, extends, import, super），以及各大浏览器自行增加的const保留字，也是不能作为变量名的。

+ 8、禁止八进制数值

正常模式下，整数数值的第一位如果是0，表示是八进制数，而在严格模式下，禁止对数值文本分配八进制值，或尝试对八进制值使用转义，否则报错：“SCRIPT1039：严格模式下不允许使用八进制数字参数和转义字符”。

``` javascript
    "use strict";
    var testoctal = 010;    // 报错
    var testescape = \010;  // 报错
```

+ 9、禁止this指向全局对象

在严格模式下，当 this 的值为 null 或 undefined 时，该值不会转换为全局对象，比如：

``` javascript
    "use strict";
    function testFunc() {
        return this;
    }
    var testvar = testFunc();
```

在正常的模式下，testvar的值为全局对象，但在严格模式下，该值为 undefined。

再比如：

``` javascript
    function f(){
　　　　return !this;
　　}
　　// 返回false，因为"this"指向全局对象，"!this"就是false
　　function f(){
　　　　"use strict";
　　　　return !this;
　　}
　　// 返回true，因为严格模式下，this的值为undefined，所以"this"为true。
```

因此，使用构造函数时，如果忘了加new，this不再指向全局对象，而是报错。

``` javascript
    function f(){
　　　　"use strict";
　　　　this.a = 1;
　　}
　　f();// 报错，this未定义
```

需要注意的是在IE10的PP2版（支持严格模式）中对于严格模式下得this存在bug，对于`(function(){ "use strict"; return !this })()`返回的是false。

+ 10、禁止“eval”用作标示符（变量、函数名、参数名等）

+ 11、eval具有独立作用域

正常模式下，eval语句的作用域，取决于它处于全局作用域，还是处于函数作用域。严格模式下，eval语句本身就是一个作用域，不再能够生成全局变量了，它所生成的变量只能用于eval内部。

``` javascript
    "use strict";
    var indirectEval = eval;
    indirectEval("var testvar = 10;");
    document.write(testVar);    // 报错，testVar未定义
```

+ 12、禁止在语句或块内声明函数

严格模式只允许在全局作用域或函数作用域的顶层声明函数。也就是说，不允许在非函数的代码块内声明函数，否则将会报错：“SCRIPT1047：在严格模式下，函数声明无法嵌套在语句或块中。 它们只能显示在顶级或直接显示在函数体中。”

``` javascript
    "use strict";
    var arr = [1, 2, 3, 4, 5];
    var index = null;
    for (index in arr) {
        function myFunc() {};   // 报错
    }
```

+ 13、禁止“arguments”用作标示符（变量、函数名、参数名等）

+ 14、禁止对arguments对象赋值

+ 15、arguments不再影响和追踪参数值

``` javascript
    function f(a) {
　　  "use strict";
　　  a++；
　　  return [a, arguments[0]];
　　 }
　　f(1); // [2,1]
　　
　　function f(a) {
　　  "use strict";
　　  arguments[0]++；
　　  return [a, arguments[0]];
　　 }
　　 f(1);    // [1,2]
```
在严格模式下，arguments对象只是一个本地副本。

+ 16、禁止使用arguments.callee

``` javascript
    "use strict";
　　var f = function() { return arguments.callee; };
　　f(); // 报错
```

+ 17、禁止使用with

``` javascript
    "use strict";
　　with (Math){
        x = cos(3);
        y = tan(7);
    }
```

