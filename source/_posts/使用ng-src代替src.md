title: 使用ng-src代替src
date: 2014-12-03 02:06
categories: web
---
今天调一个页面，碰巧开着console，偶然发现无效的http请求，提示404错误
<!--more-->

虽然不影响功能，还是检查了一下，发现是因为有类似下面的代码：

```
<img src="{{item.imgPath}}" alt=""/>
```

所以，在angular实际渲染完成之前，浏览器读到img标签，就向{{item.imgPath}}这个地址发起了请求，结果当然是404。实际上应该使用：

```
<img ng-src="{{item.imgPath}}" alt=""/>
```

就会有延迟加载的效果了，因为ng-src不是HTML标准attr，浏览器无法识别，也就不会发起无效的http请求