title: 设置mysql支持emoji
date: 2015-06-29 18:22
categories: database
---
emoji在诞生之初有多种标准，所以早期兼容性是个问题。但是现在已经标准化了，是unicode的一部分。可以认为，跟字母、汉字一样，emoji就是unicode中一个普通的字符。本文介绍让mysql支持emoji的方法
<!--more-->

## 什么是emoji

emoji虽然也是字符，但是通过utf-8编码后，每个字符占4个字节，属于宽字符。而老版本的mysql只支持一个字符占3个字节，所以老版本的mysql是无法存储emoji的

新版本的mysql增加了字符集utf8mb4，可以支持单字符最多占4个字节。utf8mb4是utf8的超集，可以无需修改地支持原来的utf8字符

要让mysql存储emoji，需要满足2个条件：

1、mysql的charset设置为utf8mb4

2、客户端连接mysql的驱动，也需要设置为utf8mb4

## mysql设置charset为utf8mb4

1、需要设置数据库实例的character_set_server参数为UTF8mb4

2、设置数据库字符集为utf8mb4

3、设置表的字符集为UTF8mb4

4、如果不设置表的字符集为UTF8mb4，也可以设置单独某个列的字符集为UTF8mb4

对于新开发的应用，建议都把数据库的字符集设置为UTF8mb4，以免后期迁移的麻烦

## 客户端连接mysql驱动

以node.js为例，需要设置charset参数为UTF8MB4_GENERAL_CI，全大写

其他平台如java，php，也需要做类似的配置

至于最终呈现的地方，包括html页面、iOS客户端，经实验发现，不需要特殊的设置，自动可以输入和展示emoji字符

## 不支持emoji的客户端

有些客户端不支持emoji，或者支持得不充分。我实验了一下：

navicat for mysql完全无法正确展示emoji字符

webstorm也无法完全正确展示，部分emoji字符被截断而展示不全

CocoaRestClient可以完美展示，借助复制粘贴，也可以输入emoji字符