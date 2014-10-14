---
layout: post
title: "cookie与session"
date: 2013-08-03 11:11
comments: true
categories: js 前端
---

## Cookie

HTTP Cookie，通常直接叫做cookie，是在客户端用户存储会话信息的。cookie存储的信息是由服务器经HTTP头作为响应发给客户端的。例如服务器相应可能如下：

    HTTP/1.1 200 OK
    Content-type:text/html
    Set-Cookie:name=value

其中的`Set-Cookie`包含着会话信息，浏览器会存储这些会话信息，之后会为每个请求添加Cookie HTTP头，将信息发送回服务器，如下

    GET /index.html HTTP/1.1
    Cookie:name=value

因为Cookie是存储在浏览器上，浏览器资源有限，所以Cookie的个数和大小会有限制，不同的浏览器对Cookie 的限制不同，有的限制每个域只能有20个Cookie，且每个域Cookie总大小限制在4095B，有的浏览器可能会多些。

### Cookie构成

+ 名称：唯一确定cookie，名称是不区分大小写的
+ 值：存储在cookie中的字符串的值，必须被URL编码
+ 域：表示cookie对哪个域是有效的，所有向该有效域发送的请求都会加上这个cookie信息。如果没有设置，默认是设置cookie的域
+ 路径：对于制定域中的那个路径，应该向服务器发送cookie
+ 失效时间：表示cookie应该何时被删除。如果不设定，默认情况下，浏览器会话结束就会删除cookie（会话cookie）。如果设置了时间（大于当前时间），则浏览器关闭后cookie依然会保存在用户机器上（持久cookie）。这个值是GMT格式的日期。
+ 安全标志

### Javascript操作cookie

 `document.cookie`属性返回当前页面可用的（根据cookie的域、路径、失效时间和安全设置）所有cookie的字符串，一系列由分号分隔的名值对，如下：

    name1=value1;name2=value2;name3=value3

所有名字和值都是URL编码的，所以必须使用`decodeURIComponent()`解码。

设置值得时候，可以直接给`document.cookie`属性赋值，并不会覆盖cookie，除非设置的cookie名称已经存在。

由于cookie操作比较麻烦，并不直观，下面来看一个简化cookie操作的功能对象：

```javascript

    var CookieUtil={

        get: function(name){
            var cookieName = encodeURIComponent(name)+"=",
                cookieStart = document.cookie.indexof(cookieName);
                cookieValue = null;

            if(cookieStart > -1){
                var cookieEnd = document.cookie.indexof(";",cookieStart);
                if(cookieEnd == -1){
                    cookieEnd = document.cookie.length;
                }
                cookieValue = decodeURIComponent(document.cookie.substring(cookieStart+cookieName.length, cookieEnd));
            }

            return cookieValue
        }

        set: function(name, value, expires, path, domain, secure){
            var cookieText = encodeURIComponent(name)+"="+encodeURIComponent(value);

            if(expires instanceof Date){
                cookieText += "; expires=" + expires.toGMTString();
            }

            if(path){
                cookieText += "; path=" + path;
            }

            if(domain){
                cookieText += "; domain=" + domain;
            }

            if(secure){
                cookieText += "; secure";
            }

            document.cookie = cookieText;
        }
    }

```

<!-- more -->

## Session

Session机制是一种服务器端维持会话状态的机制。每个session唯一的由一个session id标示，session id是存储在客户端的，由客户端请求时发给服务器来与相应的session建立联系。

session id一般来说是保存在cookie中，当然如果cookie被禁用了，我们就需要其他的存储手段，比如：URL重写，就是把session id附加到URL路径的后面，还有一种技术叫表单隐藏字段，给表单添加一个隐藏字段。

### Session什么时候创建

session是在服务器端程序调用HttpServletRequest.getSession(true)这样的语句时才被创建，而不是客户端访问的时候被创建的。

注意JSP如果没有显示的关闭session，会自动创建session。

### Session什么时候被删除

+ 程序调用`HttpSession.invalidate()`
+ session超时
+ 服务器进程停止

要注意的是session id一般是保存在会话cookie中，会话cookie会在浏览器关闭之后被删除，于是，相应的session id也会被删除，但是这时并不代表session也被删除了，服务器会一直保留session直到session处于非活动状态的时间超过了设置的超时时间，这时才会删除session。

## Cookie与Session的区别

cookie机制采用的是在客户端保持状态的机制，cookie信息保存在客户端。cookie存储空间小。

session机制采用的是服务器端保持状态的机制，session信息保存在服务器端。
