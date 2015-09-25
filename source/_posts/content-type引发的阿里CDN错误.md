title: content-type引发的阿里CDN错误
date: 2015-06-04 21:22
categories: web 
---
最近发现一个问题，同样一个视频，在深圳可以打开，在南京地区的android可以打开，南京地区的iPhone无法打开。本文记录一下问题解决的过程
<!--more-->

我们的视频都是通过CDN下发的，把链接改成资源实际所在的地址，绕开CDN，就都可以正常访问了，所以定位出是CDN的问题

可能是深圳的CDN节点是好的，南京的android设备和iOS设备访问了不同的节点，而iOS节点有错误

联系了CDN提供商，对方的技术人员也没有定位出问题，最后检查发现，资源的content-type没有设置，是默认的plain/text，改成video/mp4就可以了

这个问题比较特殊，虽然修改了content-type以后解决了，但是至今也不知道为什么个别CDN节点对这个http header敏感，另外一些节点又不会有问题