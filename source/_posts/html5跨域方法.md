title: html5跨域方法
date: 2015-06-13 21:50
categories: web
---
从html5开始，可以通过在响应头里增加Access-Control-Allow-Origin，实现跨域请求
<!--more-->

# 用Access-Control-Allow-Origin实现跨域

node的代码：

```
res.setHeader('Access-Control-Allow-Origin', '*');
res.setHeader('Access-Control-Allow-Credentials', true);
res.setHeader('Access-Control-Allow-Methods', 'POST, GET, PUT, DELETE, OPTIONS');
res.setHeader('Access-Control-Allow-Headers', 'Authorization,Origin, Accept, Content-Type, X-HTTP-Method, X-HTTP-METHOD-OVERRIDE,XRequestedWith,X-Requested-With,xhr,x-devicetype');
```
当然，生产环境里应该设置成允许跨域访问的源站的域名。Allow-Headers是允许跨域请求携带的http request header

客户端的代码：

```
$.ajax({
                    type: 'POST',
                    url: url,
                    data: {
                        content: content
                    },
                    beforeSend: function(request) {
                        request.setRequestHeader("x-wxopenid", hash);
                        request.setRequestHeader("x-devicetype", deviceType);
                    },
                    xhrFields:{
                        withCredentials: true
                    },
                    crossDomain: true,
                    success: function(data){
                        showTipsThenClose();
                    },
                    error: function(err){
                        showTipsThenClose();
                    },
                    dataType:"json"
                });
```

# 传统的方式

如果不借助这个新的header的话，也有传统的方式可以“跨域”

通过

```
<img src="">
<script src="">
```
等标签，可以实现跨域发送GET请求

通过

```
<form action="" method="POST">
```
可以实现跨域发送POST请求

但是这种方法不像AJAX，不能对response进行处理。但是也经常被用于CSRF攻击