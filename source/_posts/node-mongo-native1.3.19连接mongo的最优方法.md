title: node-mongo-native1.3.19连接mongo的最优方法
date: 2013-10-07 20:31
categories: javascript 
---
最近需要在node下连接mongo，尝试了很多方法，本文简要总结一下
<!--more-->

# 选择Driver

首先，基本上有4个常见的driver供选择

1、官方的是node-mongo-native

2、基于node-mongo-native，封装的mongoose，是一个ODM小框架

3、kiss小组同样基于node-mongo-native封装的mongoskin

4、mongojs

mongoose要求使用Document Schema，我们目前没有这个需求，所以不想用；mongoskin网上评价还可以，但是其GitHub库很久没更新了，而且查看了源代码，发现它是基于很老版本的node-mongo-native封装的，底层用的API现在都不推荐了，担心如果官方驱动继续升级，mongoskin没人维护；mongojs没怎么了解。总之最后还是决定用官方的node-mongo-native作为driver

# 重要文档

以下是node-mongo-native的相关文档：

[官方Manual](http://mongodb.github.io/node-mongodb-native/)

[Mongo上该Driver的主页](http://docs.mongodb.org/ecosystem/drivers/node-js/)

[官方GitHub](https://github.com/mongodb/node-mongodb-native)

其中比较重要的文档：

[how to connect in a new and better way](http://mongodb.github.io/node-mongodb-native/driver-articles/mongoclient.html)

[MongoClient API](http://mongodb.github.io/node-mongodb-native/api-generated/mongoclient.html)

# 连接方式

截止到本文，driver的最新版本是1.3.19，由于要向后兼容，所以旧的API只是不推荐使用，并没有删除，介绍的文档又比较少，所以刚上手的时候会比较迷惑应该用哪个API

在1.2版本之前，是通过Db这个对象来连接，从1.2版本开始，推荐使用MongoClient这个对象

MongoClient并不是node driver自创的，而是mongodb官方推荐的新的连接方式，可以认为是一种规范，各语言的driver是规范的实现。node-mongo-native就是这种规范的一种实现（自1.2版）

[Release Note: Default Write Concern Change](http://docs.mongodb.org/manual/release-notes/drivers-write-concern/)

使用MongoClient也有2种方式，一种是使用

```
var MongoClient = require("mongodb").MongoClient;
var client  = new MongoClient();
client.open()
client.close()
client.db()
```
这几个MongoClient的实例方法，构造方法还涉及到Server、ReplSet、Mongos等，比较繁琐。作者已经不推荐使用了：

[deprecate direct Db/Server/ReplSet/Mongos](https://github.com/mongodb/node-mongodb-native/issues/1013)

![](http://img.blog.csdn.net/20131007201038265?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva3lmeGJs/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

所以目前官方推荐的做法，是使用Connect URI + MongoClient.connect()方法

示例代码：

```
var mongoClient = require('mongodb').MongoClient;

var url = "mongodb://localhost:2222,localhost:3333,localhost:4444/mydb?maxPoolSize=10&w=1&journal=true";

// Open the connection to the server
mongoClient.connect(url, function (err, db) {

    db.collection("test", {}, function (err, collection) {
        collection.count(function (err, count) {
            console.log("there are " + count + " documents in the collection");
            db.close();
        });
    });

});
```

使用的是connect url，然后用MongoClient.connect(url, option, callback)函数来连接。上面的例子，在url中指定了dbname，那么会直接创建到目标db的连接。<span style="color:#ff0000">如果省略dbname，则是创建到admin db的连接，而不是缺省的test db，这和shell的行为不一样</span>

实际上第二个参数option经常是被省略的，option的作用是，某些参数如果在url中没有提供，那么可以在option中指定，或者在option中覆盖url中的配置

回调函数第一个参数是error，第二个参数类型不是MongoClient，而是Db，这是和new MongoClient().open()函数的主要区别

这种方式应该是目前的最佳实践，关键是如何配置连接url，在上面那个[how to connect in a new and better way](http://mongodb.github.io/node-mongodb-native/driver-articles/mongoclient.html)里描述得非常清楚了，需要时可以查看