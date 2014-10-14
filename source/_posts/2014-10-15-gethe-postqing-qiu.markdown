---
layout: post
title: "GET和POST请求"
date: 2013-08-05 11:12
comments: true
categories: 前端
---

### GET请求

GET是最常见的请求类型，最常用于向服务器查询信息。如果要发送信息给服务器的话，需要将查询字符串参数追加到URL的末尾。

使用GET请求经常会发生一个错误，就是查询字符串参数的格式有问题。对于Javascript来说查询字符串中每个参数的名称和值都必须使用`encodeURIComponent()`进行编码，然后才能放到URL的末尾。

### POST请求

POST通常用于向服务器发送应该被保存的数据。POST请求把数据作为请求的主体提交。POST请求的主体可以包含非常多的数据，而且格式不限。

### GET和POST的区别

+ GET只要是用来查询数据；POST主要用来提交数据
+ GET提交的数据是缀到URL末尾的，GET方式值发送HTTP消息头，没有消息体；POST的提交的数据是在消息体中
+ GET提交的数据量小，因为URL长度有限制；POST能提交大量数据
+ GET消耗的资源少；POST消耗的资源多
