title: http协议简单总结
date: 2013-09-24 11:06
categories: web 
---
http小总结
<!--more-->

本文大量参考[http](http://zsxxsz.iteye.com/blog/568250)，对原作者表示感谢 

# 协议层次 

http是应用层的协议，地位类似于SMTP FTP等，是构建在传输层协议TCP之上 

![](http://dl.iteye.com/upload/attachment/0071/8427/9b009ed5-bc55-3d04-8a93-22adbe1091eb.jpg)

# 数据封装 

由于http处于最上层的应用层，所以其HTTP报文需要经过多次封装，才能在网络间传递 

![](http://dl.iteye.com/upload/attachment/0071/8435/d6e37726-53d1-364b-9ac5-7f1e432378ac.jpg)

# 消息格式 

![](http://dl.iteye.com/upload/attachment/0071/8431/0d39ab7c-6b7f-37e4-a337-f5e7d3ddc458.jpg)

HTTP请求和HTTP响应，都称为http消息，包括消息头和消息体 

消息头包括消息行、实体头、头部结束标识；其中实体头又包括通用头、请求头（响应头）、实体头 

消息体就是HTTP数据体，在RFC2616中称为HTTP Entity，不是必选的。比如HTTP GET请求，就只有请求头，没有请求体 

# 消息格式举例 

![](http://dl.iteye.com/upload/attachment/0071/8433/7a1da177-6374-3c94-b475-f2650350225a.jpg)

# HTTP的局限性 

http规范的几个约束，决定了它目前有一些缺陷，需要通过其他方式来进行规避 

首先http协议是无状态的，所以每次请求对于服务端来说都是新请求，服务端无法记忆客户端此前发来的请求。规避这个限制的方法有session cookie等，都需要额外编程来实现 

其次http协议是text-based的，也就是说http消息体里面，都是一些纯文本，没法天然表示对象的类型（json某种程度上就可以）。所以在服务端，需要额外的编程，来把纯文本信息转换成对象 

最后，http协议是纯粹的请求-响应模型，也就是说，服务端没有办法主动发起对客户端的请求。 规避这个问题，最常见的方式是客户端用轮询的方式，定时主动发起请求，给人的感觉就像是服务端主动推送的一样。但是如果轮询太频繁，对服务端会造成较大压力 

还有一种方式，是服务端和客户端保持长连接，服务端利用这个长连接，将后续的消息发回客户端，这类实现有comet等 

另外，现在的html5，有一种websocket，也能解决这个问题 

还有一种方法，是在客户端添加某些插件，比如flex，以及古老的applet 

总的来说，我见过最多的还是第一种方式，comet和websocket，现在好像都还不是很成熟，没见过广泛的应用

# 与socket的关系 

socket在不同的平台有不同的实现，比如java.net包里，就提供了java平台的socket实现，利用这组API，可以进行比较容易的网络编程。 socket用的不是http协议，只是其传输层协议也是用的TCP和UDP。总的来说，socket和http关系不大。http是应用层协议，而socket是一组编程API接口。两者都是构建于传输层协议（TCP、UDP）之上