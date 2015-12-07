title: 下载tomcat时的一个细节问题
date: 2013-09-24 11:14
categories: java 
---
下载带ARP支持的tomcat
<!--more-->

在apache的tomcat下载页面上，可以看到有2个不同的链接 

![](http://dl.iteye.com/upload/attachment/0075/8361/1f623be7-4e46-310d-b12b-d8c046b175c8.png)

一个叫zip，另一个叫32-bit Windows zip 

今天偶然发现这2个链接其实有区别，32-bit Windows zip这个链接下载下来之后，已经自带了APR的库 

解压缩以后看一下： 

![](http://dl.iteye.com/upload/attachment/0075/8364/f4bdc5eb-0bd6-3357-b59e-92d268136d2f.png)
![](http://dl.iteye.com/upload/attachment/0075/8366/2c39bbc9-2314-3cfb-9579-e1d996836ac2.png)

可以看到，32-bit Windows zip解压之后，多了几个文件 

启动时就可以看到明显区别： 

![](http://dl.iteye.com/upload/attachment/0075/8369/70e2f29b-d789-3e58-b600-1a8cb10d2a30.png)
![](http://dl.iteye.com/upload/attachment/0075/8371/8857675f-c309-3b10-93f3-addca2482a35.png)

所以，如果需要启动APR支持的话，应该下载32-bit Windows zip；或者可以下载普通的zip版，然后再另外单独下载APR的支持库