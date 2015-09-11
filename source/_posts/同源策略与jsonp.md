title: 同源策略与跨域请求
date: 2014-02-23 20:22
categories: web 
---
本文简要介绍同源策略和跨域请求
<!--more-->

# 同源策略

浏览器对运行其中的javascript代码设置了诸多安全性限制，其中包括同源策略

简单来说，就是默认情况下，来自于A源的js代码，无法访问B源的document，也无法向B源发送ajax请求

源的定义包括协议，主机，端口号，比如www.example.com和www.example.com:8080就是不同的源。因此，www.example.com/index.html加载的index.js中的代码，默认情况下不能向www.example.com:8080/users发送ajax请求，但是可以发往www.example.com/users

重要的是，一段js代码属于哪个源，___并不是看它是从哪里加载的，而是看它是被谁加载的___

举个例子，很多页面引用angular，jquery等js框架时，并不从自己的服务器上获取js文件，而是从公共的CDN上获取。比如说某网站www.abc.com，使用angular作为前端MVC框架，那么它的angular.js来自google CDN

```
<script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.2.13/angular.min.js"></script>
```
这种情况下，angular.min.js虽然来自ajax.googleapis.com，但是它是属于www.abc.com这个源，可以往此服务器上的REST服务发送ajax请求，但是反而不能往ajax.googleapis.com上发送请求

# jsonp

但是在现实情况中，跨域请求是一个常见的需求。比如news.abc.com和games.abc.com虽然是不同的源，但是是同一个域的不同子站点，互相之间的通信被允许是合情合理的。这种场景下，就需要有办法来克服同源策略的限制

现在的方法有很多种，包括html5里的postMessage，Access-Control-Allow-Origin等。jsonp也是其中的一种方法

jsonp的原理，是通过把请求构造在script的地址中，其中拼接好目标服务的URL和参数，比如：

```
<script src="http://games.abc.com/user/123456"></src>

```
浏览器认为这是一个普通的js加载请求，而不是一个ajax请求，于是就会向src发起http get请求，而在games.abc.com，这个服务返回的响应是一段js代码，于是当news.abc.com的js代码得到响应之后，就可以取出代码并执行

业界的通常做法，这个响应里包含的是请求发起页面的js方法名，以及用来表示数据的json，请求方对这段json进行解析（parse），因此这种跨域请求的做法，被称为jsonp

# chrome extension的特例

利用chrome.tabs.executeScript()方法，可以做很多事情，当然也包括获取document里的信息，比如说：

```
function click(e) {
	chrome.tabs.executeScript(null, {
		code : "document.body.style.backgroundColor='red'"
	});
	window.close();
}
```
上述js代码，在点击chrome扩展中的一个按钮时，会改变当前TAB页面里的背景颜色 

看起来，chrome extension违反了同源策略，但是实际上并不是这样，chrome通过2个层面来保证安全

首先chrome扩展是需要通过google审查的，并不是随意发布的，所以比来路不明的javascript安全 

其次，在开发chrome extension时，需要在manifest文件中进行权限声明：
```
"permissions": [
    "tabs",
    "notifications",
    "http://localhost:8080/",
    "http://www.baidu.com/"
]
```

在用户选择安装这个extensions时，就会给出警告，类似于android上的权限声明机制 

所以，chrome extensions并不是违反javascript同源策略的 

# chrome扩展和插件（extensions and plugins） 

chrome的扩展和插件不同 

扩展主要用javascript开发，可以用chrome://extensions管理 

插件用NPAPI开发，语言是C++，可以用chrome://plugins管理

![](http://dl.iteye.com/upload/attachment/0081/3387/090b3054-3a84-336c-8f51-79657eb2069d.png)