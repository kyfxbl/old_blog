title: 松鼠书读书笔记（五）——cookie
date: 2013-09-24 11:56
categories: web
---
http是请求响应模型，所以最初http server几乎没有信息可以判断请求来自哪个client。为了解决这个问题，就需要client识别技术。client识别技术，包括HTTP首部扩展、IP地址跟踪、fat url等，但这些方法都有缺陷，现在也用得很少了。所以本文不介绍了，只关注主流的cookie技术 
<!--more-->

# cookie的本质

cookie就是client在发送请求的时候，会额外发送一些键值对到server，这样server读取了这些信息，就可以识别client了 

server给回响应的时候，会带一个set-cookie或者set-cookie2的首部；之后client再发送请求的时候，就用cookie首部，把cookie信息发送到server去

```
GET /index.html HTTP/1.0
Host:www.abc.com
```

```
HTTP/1.0 200 OK
Set-cookie:id="123";domain="abc.com"
```

```
GET /index.html HTTP/1.0
Host:www.abc.com
Cookie:id="123"
```

# cookie的类型

Cookie的类型，有会话cookie和持久cookie。会话cookie就是关闭浏览器以后就没有了，持久cookie则会在硬盘上保留一段时间 

![](http://dl.iteye.com/upload/attachment/0079/5357/180db7a5-6ea9-36cf-845b-646cd972bbb7.jpg)

比如说这个图，就是一个会保留2个星期的持久cookie 

# cookie与session的关系

cookie里可以放很多信息，但是这样一个问题是不安全；另一个问题是每次http请求，需要发送太多cookie，会增加网络流量。所以一种常见的措施是，在server开辟一个空间，把信息保存在server端，cookie只交换一个ID作为标识。这也就是所谓的session 

session是保存在server的，所以比cookie会安全一点，同时也省一些流量。但是session的问题在于，会增加server端的负担。而且如果是集群的话，不同的server之间怎么做session同步也是一个问题 

如果客户端支持使用cookie的话，session id通过cookie传递就可以了；如果客户端禁用cookie，就需要通过其他方式来传递session id，比如通过url改写的方式 

# cookie规范

cookie规范在rfc2965里，正式名称是HTTP State Management Mechanism（HTTP状态管理机制） 

除了server端要遵循规范的要求，支持cookie以外，客户端（通常是浏览器）也要遵守规范，cookie才能正常运作。如果浏览器不支持rfc2965，根本不能识别set-cookie和cookie首部的话，那这个浏览器也就不支持cookie了