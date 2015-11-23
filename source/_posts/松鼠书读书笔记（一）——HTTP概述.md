title: 松鼠书读书笔记（一）——HTTP概述
date: 2013-09-24 11:15
categories: web  
---
松鼠书读书笔记
<!--more-->

# URI、URL、URN 

这3个概念很容易混淆

URI：是Uniform Resource Identifier的缩写，统一资源标识符 

URL：是Uniform Resource Location的缩写，统一资源定位符 

URN：是Uniform Resource Name的缩写，统一资源名 

关系上，URI就是用来唯一标识一个信息资源，而它有两种表示方式，就是URL和URN；但是URN现在还在试验阶段，没有大规模推广，所以基本上现在所有的URI都是URL。也就是说，在目前，可以粗略认为URI=URL，无视URN 

URL的格式一般是schema://address/resource，比如
```
http://www.abc.com/special/xxx.gif
```

这个URL里，协议是http，服务器是www.abc.com，资源是/special/xxx.gif（在这里，http服务器应用，负责实现提供/special/xxx.gif这个资源的方法即可，不一定就放在special文件夹下） 

# http事务 

指的就是一次http请求 + http响应 

# http事务的过程 

在http客户端向服务端发送请求报文之前，需要用IP地址和TCP端口号，在客户端和服务端之间建立一条tcp/ip连接。在http服务端向客户端发送响应报文之后，服务端会关闭这条tcp/ip连接 

以浏览器输入一个URL
```
http://www.abc.com/special/xxx.gif
```
为例 

a) 浏览器从URL中解析截取出主机名www.abc.com 
b) 浏览器通过DNS，将www.abc.com转换成服务端ip地址 
c) 浏览器从URL中解析截取出端口号，此处没有，使用默认的80端口 
d) 浏览器建立tcp/ip连接 
e) 浏览器向服务端发送http请求报文 
f) 服务端向浏览器发送http响应报文 
g) 服务端关闭tcp/ip连接 

书里没有明确说tcp连接是由服务端还是浏览器负责断开的。但是试验了一下，用telnet向一个http服务器发送请求，在得到响应之后，会看到Connection closed by foreign host。所以认为是http服务端负责关闭连接的 

# 一些HTTP应用的概念 

proxy：位于client和server之间，负责处理和转发client的http请求，可以实现安全、突破限制等许多好处。[详细介绍](http://zh.wikipedia.org/wiki/%E4%BB%A3%E7%90%86%E6%9C%8D%E5%8A%A1%E5%99%A8) 

cache：缓存网络资源，这个很好理解，我觉得应该跟proxy有点关系，或者可以一起实现。maven私服nexus就有这个功能，可以认为是一个proxy，也是一个cache 

gateway：这个以前看RFC2616不大理解，今天就懂了，也就是一个协议转换的应用，比如一个http/ftp网关。客户端对其发起http请求，它实际通过ftp协议获取资源之后，再向客户端发回http响应 

tunnel：这个好难懂，原来也不知道是干啥的。貌似是将实际要传输的数据，封装到http报文里，来达到穿越防火墙的目的。这样的话，tunnel应该也是需要有client和server两端的，tunnel client将请求封装到http报文里，然后tunnel server从http报文里解析出请求，再将响应封装到http报文里，最后tunnel client再从报文里解析出响应 

agent：客户端代理，这个比较好理解，所有发起http请求的应用，都是client agent，最常见的当然就是各种浏览器，此外还有网络爬虫之类的，都是agent。前面说的tunnel client，按这个定义来说，也是agent了；包括proxy，对于最终服务器来说，也是agent