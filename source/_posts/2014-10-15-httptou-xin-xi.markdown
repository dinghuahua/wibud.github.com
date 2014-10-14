---
layout: post
title: "HTTP头信息"
date: 2013-07-27 11:08
comments: true
categories: 前端
---

当我们在浏览器的地址栏中输入了URL之后发生了什么？

1. 浏览器分析我们输入的地址，解析所要使用的协议为HTTP协议
2. 浏览器向DNS请求解析URL的IP地址
3. 域名系统DNS解析出所请求的URL服务器的IP地址
4. 浏览器与服务器建立TCP连接
5. 浏览器发出HTTP请求
6. 服务器通过HTTP响应把文件返回给浏览器
7. 浏览器将文件进行解释，并将web页显示给用户

### HTTP头信息

每个HTTP请求和响应都分为消息头和消息体两部分，而头部信息是一定会有的。

默认情况下HTTP请求会发送下列的头信息：

+ Accept：浏览器能够处理的内容类型--content-type
+ Accept-Charset：浏览器能显示的字符集
+ Accept-Encoding：浏览器能处理的压缩编码
+ Accept-Language：浏览器当前设置的语言
+ Connection：浏览器与服务器之间连接的类型
+ Cookie：当前页面设置的任何Cookie
+ Host：发出请求的页面所在的域
+ Referer：发出请求的页面的URL
+ User-Agent：浏览器的用户代理字符串（包括浏览器类型、操作系统信息等）

看一个例子：

    GET /index.html HTTP/1.1
    Accept:application/x-shockwave-flash
    Accept-Language:zh-cn
    Accept-Encoding:gzip
    Cookie:name=value
    User-Agent:Mozilla/4.0
    Host:localhost:8080
    Connection:Keep-Alive
