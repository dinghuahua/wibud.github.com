---
layout: post
title: "Ajax相关总结"
date: 2013-08-02 11:09
comments: true
categories: 前端
---

### 创建XMLHttpRequest对象

创建XMLHttpRequest对象，跨浏览器兼容：

```javascript

    function createXHR(){

        // 如果支持原生XHR对象
        if(typeof XMLHttpRequest != "undefined"){
            return new XMLHttpRequest();
        }
        // 兼容IE6和之前版本
        else if(typeof ActiveXObject != "undefined"){

            // arguments.callee.activeXString存放版本信息
            if(typeof arguments.callee.activeXString != "string"){

                // 可能有3中不同版本的XHR对象
                var versions = ["MSXML2.XMLHttp.6.0", "MSXML2.XMLHttp.3.0", "MSXML2.XMLHttp"],
                    i,len;

                for(i=0, len=versions.length; i<len; i++){

                    // 如果不能创建相应版本的XHR对象，会抛异常，继续检查下一个版本
                    try{
                        new ActiveXObject(versions[i]);
                        // 如果可以创建该版本的XHR对象，则记录版本
                        arguments.callee.activeXString = versions[i];
                        break;
                    }catch(ex){
                        // 没有操作
                    }
                }
            }

            return new ActiveXObject(arguments.callee.activeXString);
        }else{
            throw new Error("No XHR object available");
        }
    }
```

### 响应处理

```javascript

    var xhr = createXHR():
    // 必须在调用open()之前指定onreadystatechange事件处理才能确保跨浏览器的兼容性
    xhr.onreadystartechange = function(){
        // 请求/响应过程完成
        if(xhr.readyState == 4){
            if(httpSuccess(xhr)){
                // 响应的处理
            }else{
                // 请求/响应不成功
            }
        }
    };
    xhr.open("get", "example.txt", true);
    xhr.send(null);

    function httpSuccess(xhr){
        try{
            // 如果得不到服务器状态，且我们正在请求本地文件，认为成功
            return !xhr.status && location.protocol=="file:" ||

                    // 所有200到300间的状态码表示成功（包括200，不包括300）
                    (xhr.status >= 200 && xhr.status < 300) ||

                    // 文档未修改也算成功
                    xhr.status == 304 ||

                    // Safari 在文档未修改时返回空状态
                    navigator.userAgent.indexof("Safari") >= 0 &&
                        typeof xhr.status == "undefined";

        }catch(ex){}

        return false;
    }
```

<!-- more -->

#### 常见的status返回码

+ 403:Access Forbidden。通常是服务器上文件或目录的权限设置导致
+ 404：Object not found。请求资源不存在
+ 401：Access Denied。由于用户匿名访问使用的账号被禁用，或者没有权限访问计算机。
+ 500：Internal Server Error。服务器发生错误。

### AJAX跨域请求（CORS）

跨浏览器的CORS：

    function createCORSRequest(method, url){
        var xhr = new XMLHttpRequest();
        // 检测XHR是否支持CORS
        if("withCredentials" in xhr){
            xhr.open(method, url, true);

        // 准对IE（IE8及之后版本）
        }else if(typeof XDomainRequest != "undefined"){
            xhr = new XDomainRequest();
            xhr.open(method, url);
        }else{
            xhr = null;
        }
        return xhr;
    }

    var request = createCORSRequest("get", "http://www.test.com/index");
    if(request){
        // 请求返回会触发load事件
        request.onload = function(){
            // 操作
        };

        request.send();
    }
