title: html5的变化
date: 2013-09-24 11:56
categories: web
---
周末看了下HTML5，感觉和HTML4也没有太大区别，把一些变化总结一下 
<!--more-->

# 简化文档类型声明 

对于HTML5，文档开头只需要写
```
<!DOCTYPE html>
```

而HTML4，则要写
```
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
```

# 增加了一些内容性的标签 

如code、article等，这部分没有难度，对着参考手册写就行了 

# 删除了一些样式的标签

如font、big、center等标签被删除了，因为样式的控制应该由CSS完成  

# 增加了对多媒体资源的支持 

用video和audio标签来支持视频和音频的播放 

# 增加了绘图API 

用canvas标签来支持HTML绘图，具体的API是用javascript操作的 

# 增加了浏览器本地存储的能力 

用localStorage和sessionStorage这2个对象来支持数据的本地存储，存储空间比较大，并且避免了像cookie一样在server和client之间反复传输 

具体的操作是用javascript实现的，不过现在好像在几种主流浏览器中的实现有BUG，子域名有单独的存储空间，和HTML5规范不符合 

# 支持本地化HTML应用 

通过applicationCache支持浏览器缓存，编程也是依赖javascript实现 

效果就是把HTML相关资源（包括js、css、png）等在浏览器里缓存起来，实现离线应用 

# 支持websocket协议，实现server和client的双向通信 

通过将http协议，升级为websocket协议，支持server和client的双向通信 

可想而知，这个特性需要服务端的支持。目前tomcat已经可以支持该特性，但是用的是tomcat自己的实现 

也就是说，现在要实现这个特性，server端的代码会有差异（取决于server容器），还没有一个规范的API，所以不是很方便 

相关的规范正在制定当中，RFC6455是websocket协议的规范，与http规范一样，与具体的平台无关；JSR356是java平台的websocket规范，现在还在草案阶段，这个规范推出以后，在JAVA平台就有统一的API进行websocket的server端编程了，但是估计还要很长时间 

我感觉websocket是html5里最吸引人的，因为终于可以解决长久以来的server和client双向通信问题，但是由于规范成熟度的问题，截止到目前，仍然处于“已经可以用，但是不成熟”的阶段