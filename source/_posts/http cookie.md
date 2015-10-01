title: http cookie
date: 2013-11-06 11:55
categories: web
---
cookie是用来弥补http协议无状态的一种机制，需要浏览器和server的配合，主要是通过http头中的Cookie和Set-Cookie字段来交互
<!--more-->

浏览器每次发起请求时，会自动把跟目标域相关的cookie带上，主要通过http request header中的Cookie字段

```
Cookie:connect.sess=s%3Aj%3A%7B%22count%22%3A22%7D.nm%2FP5pzXSzOW2YK0qcPdIFxvMyM3MfHr8KNT0mOqxBw; remember=1
```

浏览器拿到这个请求头，就可以取出cookie做相应的处理。不过不同的server端代码，API都不同，比如在node.js express里，比较简单：

```
if (req.cookies.remember) {
    // xxxx
}
```

如果server希望浏览器创建或销毁Cookie，则是通过http response header中的Set-Cookie字段：

```
Set-Cookie:remember=1; Max-Age=60; Path=/; Expires=Wed, 06 Nov 2013 03:49:32 GMT
```

用node.js express实现，主要是2个API：

```
res.clearCookie('remember');
res.cookie('remember', 1, { maxAge: minute });
```

基本上，大部分的逻辑是在server编码实现的，浏览器就是透明地把跟域相关的Cookie发送出去