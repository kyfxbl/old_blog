title: MongoDB初体验
date: 2013-09-24 11:26
categories: database 
---
昨天到今天，初步研究了一下MongoDB，本文把这2天学到简单总结一下
<!--more-->

主要是对照着这个系列文章自己实践了一遍： [mongodb入门](http://www.cnblogs.com/huangxincheng/archive/2012/02/18/2356595.html) 

# 优点 

感觉使用MongoDB的好处主要是：

## 性能

插入10万条数据只用了3秒钟；创建索引后的查询效率也很高 

## 灵活

mongodb无schema，因此要增删字段都很容易，更容易实现一些对灵活性要求高的需求 

## 支持集群和分片

内建对master-slave和sharding的支持，使得做读写分离、分片都比较简单 

# 缺点

同时，也有一些比较明显的问题

## 不能回滚

误操作了会比较麻烦，比如db.person.remove()

## 不支持事务

难以保证数据一致性

## 做复杂的查询会很麻烦

关联性的数据，用mongodb查询远不如sql方便

# 基本概念 

MongoDB里首先是分db，然后每个db下有collections，collection下保存文档。可以跟RDBMS对比一下，db的概念差不多，都是数据库实例；collections类似于table，但是没有schema；文档就是table里的一行记录 

默认用mongo ip:port命令，会连上test db；如果用mongo ip:port/admin，则是连上admin db。用show dbs命令可以看到当前有哪些db

mongo的客户端同时也是一个javascript的执行环境，所以各db同时也是一个js object；用use dbname命令，可以切换db 

创建db和创建collection，都不需要额外的命令（比如create table之类的），直接插入文档，就会生成db和collection，这点比较方便 

mongod是server端的进程，mongos是做sharding时的server端进程；mongo是client，也是一个js执行环境。如果编程连接MongoDB，则driver就视为client 

# 第一印象 

第一次尝试这个命令

```
db.person.insert({name:"kyfxbl",age:29})
```

会觉得有点奇怪，不过只要理解了db.person是一个js object，insert是其上的一个方法，就很好理解了。事实上，如果输入db.person.insert，还会输出这个函数体。所以也可以这样

```
var single = {name:"kyfxbl",age:29} 
db.person.insert(single) 
```

值得一提的是，MongoDB用的是bson，跟json基本上是一回事，只是数据类型更多一点 

# 条件查询

```
// where age = 20 
db.person.find({"age":20})
```

```
// where age > 20 
db.person.find({"age":{$gt:20}}) 
```

```
// where age >= 20 
db.person.find({"age":{$gte:20}}) 
```

```
// where age > 20 and favourite = "dota" 
db.person.find({"age":{$gt:20},"favourite":"dota"}) 
```

```
// where age < 20 or favourite = "dota" 
db.person.find({$or:[{"age":{$lt:20}},{"favourite":"dota"}]}) 
```

```
// where name in "zhengshengdong","kyfxbl" 
db.person.find({"name":{$in:["zhengshengdong","kyfxbl"]}}) 
```

```
// where name not in "zhengshengdong","kyfxbl" 
db.person.find({"name":{$nin:["zhengshengdong","kyfxbl"]}}) 
```

```
// find name startwith 'k' and endwith 'l' 
db.person.find({"name":/^k/,"name":/l$/}) 
```

```
// custom query function 
db.person.find({$where:function(){return this.age > 20}}) 
db.person.find({$where:function(){return this.name == "kyfxbl"}}) 
```

其中，$where绝对是个大招啊，非常强力。不过因为是大招，所以CD也长，不能随便乱放 

# update

```
// 更新的错误方法，这行记录会只剩下"age":20 
db.person.update({"name":"zhengshengdong"},{"age":20}) 
```

```
// 正确方法 
var condition = {"name":"zhengshengdong"} 
var model = db.person.findOne(condition) 
model.age = 20 
db.person.update(condition,model) 
```

```
// $inc和$set 
db.person.update({"name":"def"},{$inc:{"age":17}}) 
db.person.update({"name":"def"},{$set:{"age":23}}) 
```

```
// upsert 
db.person.update({"name":"liting"},{"age":100},true) 
```

```
// batch update 
db.person.update(condition,{$set:{"age":99}},false,true) 
```

# 聚合和游标

```
// group 
db.person.group({ 
key:{age:true}, 
initial:{}, 
reduce:function(cur,prev){}, 
finalize:function(out){}, 
condition:{} 
})
```

```
// cursor 
db.person.find().limit(2) 
```

# 索引

索引可以有效提升查询效率 

```
db.person.find({name:"zsd10000",age:10000}).explain() 
{ 
        "cursor" : "BasicCursor", 
        "isMultiKey" : false, 
        "n" : 1, 
        "nscannedObjects" : 100000, 
        "nscanned" : 100000, 
        "nscannedObjectsAllPlans" : 100000, 
        "nscannedAllPlans" : 100000, 
        "scanAndOrder" : false, 
        "indexOnly" : false, 
        "nYields" : 0, 
        "nChunkSkips" : 0, 
        "millis" : 54, 
        "indexBounds" : { 

        }, 
        "server" : "SZXY3Z001739121:27017" 
} 
```

```
db.person.find({name:"zsd10000",age:10000}).explain() 
{ 
        "cursor" : "BtreeCursor name_1", 
        "isMultiKey" : false, 
        "n" : 1, 
        "nscannedObjects" : 1, 
        "nscanned" : 1, 
        "nscannedObjectsAllPlans" : 2, 
        "nscannedAllPlans" : 2, 
        "scanAndOrder" : false, 
        "indexOnly" : false, 
        "nYields" : 0, 
        "nChunkSkips" : 0, 
        "millis" : 34, 
        "indexBounds" : { 
                "name" : [ 
                        [ 
                                "zsd10000", 
                                "zsd10000" 
                        ] 
                ] 
        }, 
        "server" : "SZXY3Z001739121:27017" 
} 
```